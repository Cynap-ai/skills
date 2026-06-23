# Investigation Workflows

Two background `Workflow` scripts. Launch both, then stop — they notify on completion. They are **read-only**: research and design, never mutate. Adapt the inventory/paths to the current machine before launching.

Shared rules baked into every agent prompt: *"Investigation only — design the plan, do not mutate. WebFetch / read-only gh / local reads only. No CI polling, no sleep loops, no nested dispatch."*

---

## Workflow A — mine + consolidate

Mines the past week of work and surveys every harness surface, then consolidates one sequenced plan.

```js
export const meta = {
  name: 'harness-mine-consolidate',
  description: 'Mine recent sessions + survey harness surfaces, consolidate one improvement plan',
  phases: [{ title: 'Mine' }, { title: 'Survey' }, { title: 'Consolidate' }],
}

// Bucket the past-week transcripts so each miner gets a slice (find them first, inline):
//   ls -t ~/.claude/projects/*/*.jsonl | head -N   (and ~/.claude/history.jsonl)
const BUCKETS = args.transcriptBuckets // [[path,...], ...]  passed in by the skill
const LESSON_SCHEMA = { type:'object', additionalProperties:false, required:['report_markdown','lessons'], properties:{
  report_markdown:{type:'string'},
  lessons:{type:'array',items:{type:'object',additionalProperties:false,
    required:['title','category','evidence','recurrence','status','proposed_change'],
    properties:{ title:{type:'string'}, category:{type:'string'},
      evidence:{type:'string',description:'quote + session file'}, recurrence:{enum:['once','few','recurring']},
      status:{enum:['new','refines-existing','already-documented-but-still-recurring']},
      proposed_change:{type:'string',description:'exact AGENTS.md/CLAUDE.md/memory edit'} }}},
}}

phase('Mine')
const minings = await parallel(BUCKETS.map((b,i)=>()=>agent(
  `Mine these transcripts for harness lessons — extract human-typed turns, corrections, and the assistant's own self-flagged mistakes. Files: ${b.join(', ')}. Rank by impact; quote evidence with the session file. Return report_markdown + lessons[].`,
  {label:`mine:${i}`, phase:'Mine', schema:LESSON_SCHEMA, agentType:'general-purpose'})))

phase('Survey')
const SURVEY_SCHEMA = { type:'object', additionalProperties:false, required:['report_markdown','findings'], properties:{
  report_markdown:{type:'string'}, findings:{type:'array',items:{type:'string'}} }}
const surveys = await parallel([
  ()=>agent(`Survey GitHub for the period: merged PRs across the repos the user works in, the repos they touch most, and any NEWLY-CREATED repos in their orgs (gh repo list <org> --json name,createdAt). What changed that the harness should know? Return report_markdown + findings[].`, {label:'survey:github', phase:'Survey', schema:SURVEY_SCHEMA, agentType:'general-purpose'}),
  ()=>agent(`Survey Linear via MCP: recent issue comments + status changes in the active team. Surface decisions/blockers the harness rules or memory should capture. Return report_markdown + findings[].`, {label:'survey:linear', phase:'Survey', schema:SURVEY_SCHEMA, agentType:'general-purpose'}),
  ()=>agent(`Survey worktrees + memory: 'git worktree list' (prune candidates = merged/stale), and ~/.claude/projects/*/memory/ (MEMORY.md size vs limit, topic drift, uncommitted files). Return report_markdown + findings[].`, {label:'survey:wt-memory', phase:'Survey', schema:SURVEY_SCHEMA, agentType:'general-purpose'}),
  ()=>agent(`Deep-audit the harness: ~/.claude/{agents,commands,hooks,skills} + the skill-lock. Find dead pointers (skills/commands referencing missing engines/files), orphan/misfiled files, never-invoked agents (grep transcripts), and dup/conflicting commands vs native skills. Return report_markdown + findings[] with concrete delete paths.`, {label:'survey:audit', phase:'Survey', schema:SURVEY_SCHEMA, agentType:'general-purpose'}),
])

phase('Consolidate')
const PLAN_SCHEMA = { type:'object', additionalProperties:false,
  required:['executive_summary','new_lessons_ranked','prune_list','doc_edits','memory_updates','sequenced_execution','open_questions'],
  properties:{ executive_summary:{type:'string'}, new_lessons_ranked:{type:'string'}, prune_list:{type:'string'},
    doc_edits:{type:'string',description:'exact AGENTS.md + global CLAUDE.md edits'}, memory_updates:{type:'string'},
    sequenced_execution:{type:'string',description:'prune→vendor→upgrade→docs→memory, each item tagged target-repo + risk'},
    open_questions:{type:'array',items:{type:'object',additionalProperties:false,required:['question','recommendation'],
      properties:{question:{type:'string'},recommendation:{type:'string'}}}} }}
return await agent(
  `Consolidate into ONE sequenced harness-improvement plan. Dedup lessons against existing memory (only NET-NEW or doc'd-but-recurring earn a change). Honor the user's rules (never-slice, delete-legacy, Linear source-of-truth, fewer-bigger-PRs). MININGS: ${JSON.stringify(minings.filter(Boolean))}\n\nSURVEYS: ${JSON.stringify(surveys.filter(Boolean))}\n\nProduce the structured plan.`,
  {label:'consolidate', phase:'Consolidate', schema:PLAN_SCHEMA, agentType:'general-purpose'})
```

---

## Workflow B — latest + models

Research the latest version of everything GitHub-derived + audit model pins.

```js
export const meta = {
  name: 'harness-latest-upgrade',
  description: 'Latest versions of every GitHub-derived plugin/skill/CLI + model audit',
  phases: [{ title: 'Research' }, { title: 'Plan' }],
}
const COMP_SCHEMA = { type:'object', additionalProperties:false, required:['report_markdown','components'], properties:{
  report_markdown:{type:'string'}, components:{type:'array',items:{type:'object',additionalProperties:false,
    required:['name','source','installed','latest','status','update_command','risk','notes'],
    properties:{ name:{type:'string'}, source:{type:'string'}, installed:{type:'string'}, latest:{type:'string'},
      status:{enum:['current','outdated','unknown','deprecated']}, update_command:{type:'string'},
      risk:{enum:['low','medium','high']}, notes:{type:'string'} }}} }}

// INV: read ~/.claude/plugins/{installed_plugins,known_marketplaces}.json for the live inventory and pass it in.
const INV = args.inventory
phase('Research')
const [official, third, skills, models, cli] = await parallel([
  ()=>agent(`Find the latest version of each plugin from the official marketplace and the non-interactive update command (claude plugin marketplace update / claude plugin update). INV: ${INV}. Return report_markdown + components[].`, {label:'research:official', phase:'Research', schema:COMP_SCHEMA, agentType:'general-purpose'}),
  ()=>agent(`Find latest for every THIRD-PARTY marketplace/plugin (releases/tags/commits) + flag dangling marketplaces (registered, no plugin installed). INV: ${INV}. Return report_markdown + components[].`, {label:'research:third', phase:'Research', schema:COMP_SCHEMA, agentType:'general-purpose'}),
  ()=>agent(`The skills fork + any other GitHub-derived skills: latest vs installed, dedupe duplicate installs, npx skills update semantics. Return report_markdown + components[].`, {label:'research:skills', phase:'Research', schema:COMP_SCHEMA, agentType:'general-purpose'}),
  ()=>agent(`MODEL audit: read every model: pin in ~/.claude/agents/*.md, settings.json, and skill frontmatter. Flag ONLY genuinely-stale IDs vs the current flagships (latest Opus/Sonnet/Haiku/Fable + Codex); leave already-latest and intentional cheap-tier routing alone. Return report_markdown + components[] (name=file:loc, installed=current, latest=recommended).`, {label:'research:models', phase:'Research', schema:COMP_SCHEMA, agentType:'general-purpose'}),
  ()=>agent(`Is the Claude Code CLI + skills CLI + Codex CLI on latest? Give the update command for each. Return report_markdown + components[].`, {label:'research:cli', phase:'Research', schema:COMP_SCHEMA, agentType:'claude-code-guide'}),
])
phase('Plan')
const UP_SCHEMA = { type:'object', additionalProperties:false, required:['version_matrix','update_commands','model_modernization','prune_or_dedupe','risk_and_ordering'], properties:{
  version_matrix:{type:'string'}, update_commands:{type:'string'}, model_modernization:{type:'string'},
  prune_or_dedupe:{type:'string'}, risk_and_ordering:{type:'string'} }}
return await agent(`Merge into ONE ready-to-run upgrade plan: version matrix, ordered scriptable commands (backup first), model edits, prune/dedupe, risk+ordering (isolate MAJOR bumps last; flag bake-required items). Streams: ${JSON.stringify([official,third,skills,models,cli].filter(Boolean))}`, {label:'upgrade-plan', phase:'Plan', schema:UP_SCHEMA, agentType:'general-purpose'})
```

---

## Workflow C — agent-stack landscape (periodic / ~quarterly)

Survey the best/newest GitHub agent stacks and gap-analyze our owned agents. The landscape moves slowly — run on a deeper pass, not every time. **CRITICAL: live-verify every repo** (`api.github.com/repos/<o>/<r>` for `stargazers_count`/`pushed_at`/`archived`; `/search/repositories?...sort=stars`) — never trust training-data recall for stars/recency/existence, and never trust the npm package name to identify the active repo (an npm-name squatter mis-led a prior run: the real `oh-my-claudecode` is `Yeachan-Heo/...` v4.x, not the 2★ `ragingstar2063/...` that owns the npm name).

```js
export const meta = {
  name: 'agent-stack-landscape',
  description: 'Survey GitHub agent stacks, gap-analyze vs our owned agents, recommend stay/cherry-pick/adopt',
  phases: [{ title: 'Discover' }, { title: 'Recommend' }],
}
const OURS = args.ourStack // read ~/.claude/agents/*.md (names, models) + our rules; pass in
const CAND = { type:'object', additionalProperties:false, required:['report_markdown','candidates'], properties:{
  report_markdown:{type:'string'},
  candidates:{type:'array',items:{type:'object',additionalProperties:false,
    required:['name','repo','stars','last_activity','category','maturity','fit','verdict','notes'],
    properties:{ name:{type:'string'}, repo:{type:'string'}, stars:{type:'string'}, last_activity:{type:'string'},
      category:{enum:['subagent-collection','orchestration-framework','agent-os','sdk','skills','meta-list','adjacent-frontier']},
      maturity:{enum:['active','maintained','dormant','archived','unknown']}, fit:{type:'string'},
      verdict:{enum:['adopt','cherry-pick','skip','watch']}, notes:{type:'string'} }}} }}
const PIN = 'Research only. Live-verify every repo on GitHub. No mutation, no polling, no nested dispatch.'
phase('Discover')
const [subs, orch, frontier, gaps] = await parallel([
  ()=>agent(`${PIN}\nBest Claude Code SUBAGENT COLLECTIONS (native .claude/agents/*.md). Search stars-sorted + check the known ones (wshobson/agents, VoltAgent/awesome-claude-code-subagents, contains-studio/agents, 0xfurai). Are any DEEPER than ours, current-model-pinned, permissive-licensed? OURS: ${OURS}. report_markdown + candidates[].`, {label:'stack:subagents', phase:'Discover', schema:CAND, agentType:'general-purpose'}),
  ()=>agent(`${PIN}\nBest agent ORCHESTRATION frameworks (claude-flow/ruflo, SuperClaude, claude-squad, crystal, OMC v4 Yeachan-Heo). Do any materially beat our own orchestrator + dynamic Workflow engine WITHOUT re-adding a parallel architecture? report_markdown + candidates[].`, {label:'stack:orch', phase:'Discover', schema:CAND, agentType:'general-purpose'}),
  ()=>agent(`${PIN}\nOFFICIAL + frontier: anthropics/claude-agent-sdk, anthropics/skills, superpowers deltas, and adjacent (OpenAI Agents SDK, Google ADK, LangGraph, Agno) — what PATTERN is worth borrowing, harness-class vs app-runtime-class? report_markdown + candidates[].`, {label:'stack:frontier', phase:'Discover', schema:CAND, agentType:'general-purpose'}),
  ()=>agent(`${PIN}\nGAP-ANALYZE our owned agents (read ~/.claude/agents/*.md) honestly: which ROLES are missing that contradict our own rules or recur weekly (e.g. refactorer, sql/data, deployment)? Where are ours already deeper? candidates[] = gaps (name=role, repo=best source to mine). report_markdown + candidates[].`, {label:'stack:gaps', phase:'Discover', schema:CAND, agentType:'general-purpose'}),
])
phase('Recommend')
const REC = { type:'object', additionalProperties:false, required:['recommendation','landscape_md','adoption_plan','what_to_borrow'], properties:{
  recommendation:{type:'string'}, landscape_md:{type:'string'}, adoption_plan:{type:'string'}, what_to_borrow:{type:'string'} }}
return await agent(`Decide stay/cherry-pick/adopt for a solo founder under one-architecture + ownable-tooling rules. A wholesale parallel framework is almost always wrong (the OMC/ECC rejection); the realistic win is cherry-picking specific MISSING roles + borrowing prompt patterns, PORTED + owned + decoupled + re-model-pinned + domain-invariants-baked-in (a generic specialist ignorant of our invariants is net-negative). Streams: ${JSON.stringify([subs,orch,frontier,gaps].filter(Boolean))}`, {label:'stack-rec', phase:'Recommend', schema:REC, agentType:'general-purpose'})
```

---

## Launching

Read the live inventory inline first (`~/.claude/plugins/*.json`, `ls -t ~/.claude/projects/*/*.jsonl`, and for C `~/.claude/agents/*.md`), pass it as `args`, launch the Workflows, then **stop and wait** for the completion notifications. Do not poll.
