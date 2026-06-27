# Deep mode — dynamic multi-agent grounding

The escalation when one lean pass won't cover the task: a topic broad enough to need many sources and several angles, or valuable enough to justify the cost. Multi-agent fan-out runs ~15× the tokens of a single pass — reach for it deliberately, not by default. When even this isn't enough (every claim cited and cross-checked into a standalone report), use the `deep-research` skill instead.

## Shape

Three phases, each a fan-out, context flowing forward — the lean pass's `frame → research → brief`, scaled out:

1. **FRAME (internal fan-out)** — parallel reader agents map the premise's own reality: codebase, architecture/constraints, target & goal, prior art. Each claim cites a real `file:line`; a synthesis step distils them into a Problem Frame plus the sharp research questions that seed phase 2.
2. **RESEARCH (external fan-out, seeded by the Frame)** — one agent per research question, queries shaped by the real stack the Frame named. Each returns structured findings (claims with source URLs, standards, patterns, gotchas). A verify stage adversarially cross-checks load-bearing claims against an independent source.
3. **SYNTHESIZE** — merge Frame + verified findings into the brief (same sections as the lean pass), saved to `docs/investigations/`.

## Guardrails

- **Scale agents to breadth** — 2–3 for a narrow topic, 8–12+ for a broad one. Don't fan out wider than the topic earns.
- **Gate the same way** — deep mode is still *grounding*; it doesn't fire on a one-sentence diff.
- **No nested dispatch** — the workflow's agents must not spawn their own sub-agents.
- **Don't poll** — the run is background; read its result when it completes, never loop waiting on it.
- **Cap concurrency** — heavy local ops serialize on one machine-global lock; over-dispatch just queues.

## Workflow-script template

Adapt for the Workflow tool. Pass the task as `args.premise`; tune the angle and question lists to the topic. Plain JS (no TS); no `Date.now()` / `Math.random()`.

```js
export const meta = {
  name: 'investigation-deep',
  description: 'Deep grounding: fan out to frame the task internally (codebase/architecture/goal/prior-art, claims cite file:line), then research externally seeded by that frame (question × source, adversarially cross-verified), then synthesize a grounded brief.',
  phases: [
    { title: 'Frame', detail: 'internal fan-out: codebase, architecture, goal, prior art' },
    { title: 'Research', detail: 'external fan-out seeded by the frame, with adversarial verify' },
    { title: 'Synthesize', detail: 'merge into the grounded brief' },
  ],
}

const PREMISE = (args && args.premise) ? args.premise : ''
if (!PREMISE) log('WARN: no args.premise — pass the task to ground.')

// ---- Phase 1: FRAME (internal) ----
phase('Frame')

const FRAME_SCHEMA = {
  type: 'object', additionalProperties: false,
  required: ['angle', 'findings', 'open_questions'],
  properties: {
    angle: { type: 'string' },
    findings: { type: 'array', items: {
      type: 'object', additionalProperties: false,
      required: ['claim', 'evidence_path', 'confidence'],
      properties: {
        claim: { type: 'string' },
        evidence_path: { type: 'string', description: 'real file path or file:line — never a guess' },
        confidence: { type: 'string', enum: ['high', 'medium', 'low'] },
      },
    } },
    open_questions: { type: 'array', items: { type: 'string' } },
  },
}

const FRAME_ANGLES = [
  { key: 'codebase', prompt: 'Map where this task lives in THIS codebase: relevant modules/files, what already exists, the conventions in play. Use Read/Grep/Bash; every claim cites a real file:line.' },
  { key: 'architecture', prompt: 'Map the architectural constraints this task touches: relevant ADRs, platform invariants, data boundaries, the seams involved. Cite real file paths (docs/adr, CLAUDE.md, etc.).' },
  { key: 'goal', prompt: 'State the real target & goal: what success looks like, scope boundaries, constraints. Pull from the issue/spec/premise; cite where each constraint comes from.' },
  { key: 'prior-art', prompt: 'Find prior art in-repo: has this been attempted? related code, PRs, similar handlers/flows. Cite file paths.' },
]

const frames = (await parallel(FRAME_ANGLES.map(a => () =>
  agent('TASK TO GROUND:\n' + PREMISE + '\n\nANGLE: ' + a.prompt, { label: 'frame:' + a.key, phase: 'Frame', schema: FRAME_SCHEMA })
))).filter(Boolean)

const FRAME_OUT_SCHEMA = {
  type: 'object', additionalProperties: false,
  required: ['constraints', 'goal', 'research_questions'],
  properties: {
    constraints: { type: 'array', items: { type: 'string' } },
    goal: { type: 'string' },
    research_questions: { type: 'array', items: { type: 'string' } },
  },
}

const PROBLEM_FRAME = await agent(
  'Synthesize a Problem Frame from these internal findings + the task. Output the real constraints, the goal in one line, and 4-8 SHARP external research questions specific to THIS task and its real stack.\n\nTASK:\n' + PREMISE + '\n\nFINDINGS:\n' + JSON.stringify(frames),
  { label: 'frame:synthesize', phase: 'Frame', schema: FRAME_OUT_SCHEMA }
)

// ---- Phase 2: RESEARCH (external, seeded by the frame) ----
phase('Research')

const RESEARCH_SCHEMA = {
  type: 'object', additionalProperties: false,
  required: ['question', 'claims', 'standards', 'patterns', 'gotchas'],
  properties: {
    question: { type: 'string' },
    claims: { type: 'array', items: {
      type: 'object', additionalProperties: false,
      required: ['statement', 'source_url', 'confidence'],
      properties: {
        statement: { type: 'string' },
        source_url: { type: 'string' },
        confidence: { type: 'string', enum: ['high', 'medium', 'low'] },
        recency: { type: 'string' },
      },
    } },
    standards: { type: 'array', items: { type: 'string' } },
    patterns: { type: 'array', items: { type: 'string' } },
    gotchas: { type: 'array', items: { type: 'string' } },
  },
}

const VERIFY_SCHEMA = {
  type: 'object', additionalProperties: false,
  required: ['verdicts', 'note'],
  properties: {
    verdicts: { type: 'array', items: {
      type: 'object', additionalProperties: false,
      required: ['claim', 'verified', 'corroborating_urls'],
      properties: {
        claim: { type: 'string' },
        verified: { type: 'boolean' },
        corroborating_urls: { type: 'array', items: { type: 'string' } },
      },
    } },
    note: { type: 'string' },
  },
}

const QUESTIONS = (PROBLEM_FRAME && PROBLEM_FRAME.research_questions) ? PROBLEM_FRAME.research_questions : []

const researched = (await pipeline(
  QUESTIONS,
  (q, _orig, i) => agent(
    'Research this question, grounded for the task. Use WebSearch/WebFetch (load via ToolSearch "select:WebSearch,WebFetch" if not directly available) + site:-scoped queries; prefer primary sources. Every claim carries a real fetched source_url — drop any you cannot fetch. See the skill SOURCES.md for query recipes & gotchas (a site: miss is inconclusive → retry unscoped; re-issue WebFetch on cross-host redirects).\nFRAME: ' + JSON.stringify(PROBLEM_FRAME) + '\nQUESTION: ' + q,
    { label: 'research:' + (i + 1), phase: 'Research', schema: RESEARCH_SCHEMA }
  ),
  (res, _q, i) => {
    if (!res) return null
    const lb = (res.claims || []).slice(0, 3).map(c => '- ' + c.statement + ' [' + c.source_url + ']').join('\n')
    if (!lb) return { research: res, verification: null }
    return agent(
      'Adversarially verify these claims — try to REFUTE each; verified=true ONLY if an INDEPENDENT source (different domain than the given one) corroborates. Use WebSearch/WebFetch.\n' + lb,
      { label: 'verify:' + (i + 1), phase: 'Research', schema: VERIFY_SCHEMA }
    ).then(v => ({ research: res, verification: v }))
  }
)).filter(Boolean)

// ---- Phase 3: SYNTHESIZE ----
phase('Synthesize')

const BRIEF_SCHEMA = {
  type: 'object', additionalProperties: false,
  required: ['tldr', 'problem_frame', 'industry_standard', 'elevation', 'tips_gotchas', 'recommendations', 'what_to_research_next', 'sources'],
  properties: {
    tldr: { type: 'string' },
    problem_frame: { type: 'string' },
    industry_standard: { type: 'array', items: {
      type: 'object', additionalProperties: false, required: ['point', 'source'],
      properties: { point: { type: 'string' }, source: { type: 'string' } },
    } },
    elevation: { type: 'array', items: { type: 'string' } },
    tips_gotchas: { type: 'array', items: { type: 'string' } },
    recommendations: { type: 'array', items: { type: 'string' } },
    what_to_research_next: { type: 'array', items: { type: 'string' } },
    sources: { type: 'array', items: { type: 'string' } },
  },
}

const brief = await agent(
  'Synthesize the grounded brief for the task. Fill every section; each industry_standard carries a source; include only grounded claims — drop or mark "(unverified)" anything the verify step did not confirm. sources = deduped file:line + URLs.\n\nTASK:\n' + PREMISE + '\nFRAME:\n' + JSON.stringify(PROBLEM_FRAME) + '\nRESEARCH:\n' + JSON.stringify(researched),
  { label: 'synthesize', phase: 'Synthesize', schema: BRIEF_SCHEMA }
)

return brief
```
