# DEPRECATED — superseded by `Roeelfs/keel`

As of **2026-07-01**, this repo is **no longer the source of truth** for the agent skill set.

The engineering skills (and agents) have been converged into:
- **Public harness:** [`Roeelfs/keel`](https://github.com/Roeelfs/keel) — the generic, sanitized skills + agents. **Edit skills there** (`~/code/keel`), commit, push; the live `~/.claude/skills/*` symlinks resolve into the Keel clone.
- **Private overlay:** `Cynap-ai/claude-harness` → `~/.claude/skills-overlay/` — adopter-private LEARNINGS + the investigation maintenance runbook, read "if present" and never published.

This repo's `~/.agents/skills/*` install path is **bypassed** (the `~/.claude/skills/*` symlinks no longer point here). Kept for history only — safe to archive (`gh repo archive Cynap-ai/skills`).

Do **not** edit skills here. See [`reference_keel_canonical_skills`](../../.claude/projects/) in harness memory for the full convergence record.
