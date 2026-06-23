# Execution Gotchas

Hard-won traps, by phase. Read the relevant block before running that phase of step 4.

## Investigation (step 1)

- **Don't mutate the harness while the Workflows run.** Their agents read `~/.claude/{agents,skills,settings.json,plugins}`; pruning or editing mid-flight races them and splits the model pass. Let both Workflows finish, *then* execute.
- **Run two Workflows in parallel, not one giant one** — mining+consolidation is independent of latest-version+models. Both background; you get notified.

## Backup (step 4a)

- **Lean tar, not everything.** Back up `~/.claude/{agents,commands,hooks,skills,settings.json,CLAUDE.md}` + the small plugin manifests + `~/.agents` + the skill-lock. **Exclude `~/.claude/plugins/` cache** — it's hundreds of MB, regenerable, and a full tar of it hangs.
- `~/.claude` is itself a git repo → deletions there are also recoverable via git. `~/.agents` may not be → the tar is its only backup.
- **Verify the backup contains your delete targets** before deleting (`tar tzf … | grep <target>`).

## Prune (step 4b)

- **Verify each delete target exists first** (a stale plan may name a path that's already gone). Delete, then scan for **dangling references** — a command/skill that pointed at a now-deleted engine.

## Vendor cutover (step 4c)

- **`-s` repeats per skill** — `-s a -s b -s c`, NOT `-s a,b,c`. Comma-separated silently installs nothing (treated as one bad name) and just lists available skills.
- **The "PromptScript does not support global skill installation" failures are benign** — that's one agent type rejecting global installs; Claude Code installs fine. Confirm by checking the lock + symlinks, not the failure count.
- **Promote in-progress skills before installing** — `git mv skills/in-progress/<x> skills/engineering/<x>`, add to `.claude-plugin/plugin.json` + `README.md`, commit/push, *then* `npx skills add … -s <x>`.
- **Re-homing a skill deleted upstream** (no source): `cp` it into the fork and register it — never `npx skills update` it (404).
- **After cutover, verify:** every lock row's `source` == the fork, orphan-renamed dirs are gone (`diagnose`→`diagnosing-bugs`), and **no symlink in `~/.claude/skills` dangles**.

## Upgrade (step 4d)

- **Everything is scriptable** — `claude plugin marketplace update|remove` + `claude plugin update|install|enable` exist; no `/plugin` TUI needed. Only the **restart** (to apply) and the **bake** are human steps.
- **Model audit is surgical.** Bump only genuinely-stale IDs (e.g. a prior-gen Opus). Leave anything already on the current flagship, and leave **intentional cheap-tier routing** (a deliberately-cheaper model in an orchestrator/rescue path) — bumping it defeats the tiering.
- **Isolate MAJOR bumps and bake them.** A plugin major (e.g. superpowers 5→6) and the Codex CLI back skills/agents you rely on — do them last, flag a restart + re-test in a throwaway session. A global TypeScript major: hold until `npx tsc --noEmit` passes in the repo (global TS doesn't affect repo `tsc`, but don't assume).
- `setup-matt-pocock-skills` stays **dormant** (it configures GitHub/local trackers; our source-of-truth is Linear). `to-issues`/`to-prd`/`triage` are vendored but **Linear-mismatched** — keep, don't rely on them un-adapted.

## Merge (step 5)

- **Three repos, three mechanisms:** product-repo `AGENTS.md` → **PR** (merge = prod deploy → explicit go-ahead only); skills fork → push (then `npx skills update -g`); `~/.claude` → commit + push.
- **Never weaken `~/.claude`'s `.gitignore`.** Confirm `sessions/`, `history.jsonl`, `plugins/`, `shell-snapshots/` stay ignored before `git add -A`. Scan the `settings.json` diff for secrets before committing.
- Committing `~/.claude` also persists **accumulated memory from prior sessions** that was never committed — that's wanted (it makes the harness canonical), just expect more changed files than you authored.

## Light vs heavy ops

`npx skills`, `claude plugin`, `gh`, `git`, and WebFetch are **light** — they are not the machine-global heavy-lock ops (vitest/build/cdk). Run them directly.
