---
name: improve-harness
description: Periodically mine recent work + harness state into one consolidated improvement plan, grill it, then execute prune → vendor → upgrade → docs → memory and merge across the three harness repos.
disable-model-invocation: true
---

# Improve Harness

A periodic ritual that turns the last stretch of work into a **better agent harness**. It mines recent sessions for lessons, surveys every harness surface, researches what's gone stale, **consolidates** one sequenced plan, grills it, and converges the harness to a new **canonical** state — then merges that state across all three repos.

Run it like `/improve-codebase-architecture`: every week or two, or after a burst of heavy work has accumulated lessons and version drift. It is **user-invoked** — it only fires when you type it, because it mutates your live global harness.

## The three repos it touches

Every change lands in exactly one of these — name the target before editing:

- **product repo** (e.g. `cynap-monorepo`) — `AGENTS.md`/`CLAUDE.md` rule edits. Ships as a **PR**; merging is a prod-deploy-class gate (needs explicit go-ahead).
- **skills fork** (`Cynap-ai/skills`, cloned at `~/code/cynap-skills`) — vendored skills; commit + push, installed via `npx skills update -g`.
- **harness repo** (`~/.claude` → `Cynap-ai/claude-harness`) — agents, commands, hooks, global `CLAUDE.md`, settings, memory. Commit + push; **never weaken its `.gitignore`**.

## Reversibility first

This ritual mutates your live global harness. **Back up before any mutation** — that is step 4(a), and nothing destructive runs before it. Every step is individually reversible (git revert, lock restore, plugin re-pin).

## Process

### 1. Launch the investigation — two parallel dynamic Workflows

Fan the read-only investigation out into **two background Workflows** so they run while you do nothing. Author them from the templates in [WORKFLOW.md](WORKFLOW.md):

- **Workflow A — mine + consolidate.** Session miners over the **past week** of `~/.claude/projects/*/*.jsonl` + `history.jsonl` (extract human turns, corrections, self-flagged mistakes), plus surveyors of every harness surface: **GitHub** (merged PRs, the repos you work across, newly-created repos in your orgs), **Linear** (recent issue comments + status changes), **worktrees** (the in-flight registry; prune candidates), **memory** (`MEMORY.md` + topic-file drift), and a **vendoring + deep-audit** of skills/agents/commands/hooks. → one consolidated, sequenced plan with ranked new lessons + a concrete prune list.
- **Workflow B — latest + models.** Research the latest version of every **GitHub-derived** plugin, marketplace, and skill, the **CLIs** (Claude Code, Codex, `skills`), and a **model-pin audit** of every `model:` against the current flagship IDs. → a version matrix + ready-to-run upgrade plan.

Both run read-only. **Completion criterion:** both Workflows have returned their structured plans; nothing in the harness has been mutated yet.

### 2. Reconcile into one sequenced program

Merge the two plans into a single ordered program: **prune → vendor → upgrade → doc-edits → memory**. Reconcile overlaps — the model audit is authoritative over the audit's coarse "re-point everything" (bump only genuinely-stale IDs); a delete-target needs no model bump. Adopt the recommended defaults for any low-stakes open question; surface only genuine risk-class forks (e.g. a MAJOR plugin bump). **Completion criterion:** one written plan where every item has a target repo and a risk rating, and every cross-workflow overlap is resolved.

### 3. Grill the plan

Run the `/grill-with-docs` skill on the consolidated plan **before mutating anything** — stress-test it against the documented domain language and project rules, and let contradictions surface. `/spec-review` verifies an already-written spec; grilling fixes a plan authored from a misread. **Completion criterion:** the plan survives grilling, with any contradiction either resolved or recorded.

### 4. Execute, in order

Each phase has traps — read [GOTCHAS.md](GOTCHAS.md) before running it.

- **a. Back up** the harness (lean tar of touched dirs + `~/.agents` + the skill-lock; `~/.claude` is itself a git repo). Verify the backup contains the files you're about to delete.
- **b. Prune** dead config — **verify every delete target exists first**, then remove and confirm no dangling references remain.
- **c. Vendor** — fork or sync the skills fork, promote/adapt skills, then **cut the global install over** to the fork (`npx skills add <fork> -g -y -s <name>` per skill). Confirm every lock row points at the fork and **no symlink dangles**.
- **d. Upgrade** — `claude plugin marketplace update` then `claude plugin update` per plugin; CLIs; bump only **genuinely-stale** model pins; prune dangling marketplaces. Plugin updates need a **restart** to apply; flag MAJOR bumps for a **bake** you can't do mid-session.
- **e. Doc-edits** — apply the rule changes to the product repo's `AGENTS.md` (for the PR) and the global `CLAUDE.md` (harness repo).
- **f. Memory** — run the memory-consolidation pass first (keep the index lean), then write the new lesson files.

**Completion criterion:** every planned item is applied and locally verified; the only outstanding work is the human-gated restart + bake.

### 5. Merge everything, verify clean

Land the work in all three repos: the product-repo `AGENTS.md` **PR** (do not merge to main without an explicit go-ahead — merge is a prod deploy), the **skills fork** push, and the **harness repo** commit + push. Then verify each repo is clean and local == remote. **Completion criterion:** all three repos clean, the version matrix reflects the new canonical state, and the only remaining items are the restart + bake handed to the human.
