# Maintaining the investigation skill

Two things drift and need a procedure: the **gate** (calibrated against real session openers) and the **install** (the `skills` CLI is flaky on this private repo). Read this before editing the gate or reinstalling.

## A. Gate calibration & re-eval

The auto-fire gate is not hand-waved — it's calibrated against the operator's **real session openers** with a confusion matrix. The opener mix drifts as the work and the model change, so the gate must be re-eval'd, not set-and-forget.

### The mining + eval recipe

1. **Mine openers.** For every `~/.claude/projects/*/*.jsonl` (depth-1 = main sessions; deeper = subagent transcripts, exclude), take the first `type=="user"`, non-`isSidechain` record; strip injected `<system-reminder>`/command wrappers; flag resumed/tool-continuation sessions. Yields the real opener corpus (~328 at last run).
2. **Score each opener** with a fan-out of classifiers (chunk into ~16 batches), each labeling: `task_type`, `should_fire` (ground truth — does an **authoritative external source** change how to act?), `current_fire` (apply the live gate), `tuned_fire` (apply a candidate gate).
3. **Compute the matrix** in code: `true_fire / over_fire / miss / true_skip`, plus per-`task_type` breakdown. Optimize for **recall** (a miss is the costly error; over-fire is cheap) while keeping the two hard-skips intact.

### Current snapshot (2026-06-27, n=328)

| gate | catches | over-fires | **misses** |
|------|--------:|-----------:|-----------:|
| pre-calibration | 38 | 18 | 9 |
| **live (DOCS-vs-LOGS)** | 47 | **9** | **0** |

Two dominant should-skip classes: `internal-platform-debug` (~117) and `internal-authoring` (~75) — ~60% of openers, web can't help. The high-value should-fire class is `external-vendor-api` (~36, should-fire ~35).

### The 9 watched over-fires (under the live gate)

These fire but web grounding adds little — acceptable for an overboard operator, but **watch this set for growth** (a 10th of a *new* shape means the gate needs another carve-out):

1. `ba26e66c` — "vercel deploys fail (we switched to pnpm)" — our deploy, build logs first
2. `c190461f` — "design a portal entity-detail deep-link page" — connector contract already in-repo
3. `1b1bbee7` — "Bake F5/CYN-326 go-live-anchor billing" — Paddle named, but a runtime bake
4. `52cb716f` — "high-end presentation via notebookLM-mcp" — MCP is a delivery tool, not docs
5. `2a5282f6` — "Roam subscribeWebhook sends wrong payload" — contract already confirmed live
6. `1e4bd33a` — "CAYP Open-API handler crashes (zod)" — our sandbox bundler
7. `86d4ecf1` — "platform used to be BYOK but we…" — our credit/COGS architecture
8. `bb98e4b5` — "HANDOFF — CAYP Paddle checkout (resume)" — IDs resolved, operating our state
9. `a3b1db52` — "CYN-402 onboarding email plane" — Resend constraint already stated

### Activation rubric (extend before re-eval)

`{ skills:["investigation"], query, expected_behavior:"fires"|"skips" }` — keep ≥3 fire / ≥3 skip / ≥3 boundary:

- **fires**: "add WriteUpp's new v2 pagination cursor — never used their v2 endpoint"; "is `better-auth` cookie-cache still the recommended session strategy?"; "Paddle.js checkout init contract for the staging environment token".
- **skips**: "rename `verifySent` → `verificationSent` across the invoice handler"; "why did our reconciler heal at zero demand?" (CloudWatch); "update the CYN-xxx Linear comment".
- **boundary** (the hard cases): "fix our Turso backup parity bug" (names Turso → still SKIP, our Lambda); "Roam webhook payload is wrong" (vendor contract → FIRE); "GHA spend is 3k in 3 days, research why" (our workflows → SKIP).

### Cadence

Re-run the eval **after any gate edit**, **when the opener mix shifts materially**, and **on a model upgrade / knowledge-cutoff change** (the "familiar-but-stale-API" cell moves with the cutoff). 

**Headless-recall trap:** skill auto-trigger has recall ~0% under `claude -p` headless mode (anthropics/claude-code#36570). An *activation* eval MUST force the Skill tool (`--allowedTools Skill --output-format stream-json`, parse `tool_use` blocks named `Skill`) or it falsely reports 0% activation and every recalibration reads as a false regression.

## B. Install / update / desync runbook

The skill exists in two places: **source** (`cynap-skills/skills/engineering/investigation/`) and **installed** (`~/.claude/skills/investigation` → symlink → `~/.agents/skills/investigation/`, tracked in `~/.agents/.skill-lock.json`). They can **silently desync** — always confirm an edit reached the *installed* copy before testing the trigger.

**The `skills` CLI is unreliable for this repo.** Cynap-ai/skills is **private**, so a global `npx skills` install lands an **empty `skillFolderHash`** (`fetchSkillFolderHash()` 404s without a token), and the update path then **silently skips empty-hash entries** (vercel-labs/skills#436); `update`/`add -s` proved flaky (one `add` dumped the catalog instead of installing).

**Reliable edit → live loop:**
1. Edit in source; commit; push; merge to `main` (the lock pulls the default branch).
2. Manual install: `cp -R <src>/investigation ~/.agents/skills/investigation` and ensure `~/.claude/skills/investigation -> ../../.agents/skills/investigation`.
3. Rebuild the lock entry — `skillFolderHash` **== the git tree hash of the folder**: `git -C <repo> rev-parse HEAD:skills/engineering/investigation`. Set `source: Cynap-ai/skills`, `skillPath: skills/engineering/investigation/SKILL.md`, `pluginName: mattpocock-skills`.
4. Verify the installed `SKILL.md` description matches source, and the symlink resolves.

**Pre-install lint:** the description has a **1024-*byte*** cap (UTF-8 — em-dashes/arrows are 3 bytes each; count bytes, not code points). Run `npx skills-ref validate ./investigation` (or check `awk 'NR==3{sub(/^description: /,"");printf "%s",$0}' SKILL.md | wc -c`) before install.
