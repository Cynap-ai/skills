---
"mattpocock-skills": patch
---

Harden the **`investigation`** skill from a deep-mode self-investigation (digesting current 2026 agent-skill and research-harness guidance, adversarially verified).

- **SKILL.md** — trim the description under the 1024-**byte** cap (em-dashes are 3 bytes each — it was silently over at 1027); add a cutoff-anchored "currency self-check" tie-breaker to the gate body (familiarity ≠ currency); make the brief-save robust (`mkdir -p` + freshness header + supersede-on-slug); point at the new `MAINTAINING.md`.
- **DEEP-WORKFLOW.md** — defensive `args` parse + hard-fail on empty premise + clamped fan-out width `N` (the observed args-as-string / N-undefined failure); a four-part research-agent contract (objective/output/sources/boundaries); and verify verdicts now **enforced in code** (verified-vs-untrusted partition + URL-liveness) instead of prose-only; harness-globals declared.
- **SOURCES.md** — mark the lean-default trio vs deep-only; add `llms.txt`-first; sharpen the gotchas (GitHub index caps, WebFetch same-host redirect + JS-render limit, WebSearch's false US-only note).
- **MAINTAINING.md** (new) — the gate re-eval recipe + the 9 watched over-fires + activation rubric + cadence (incl. the `claude -p` headless-recall trap), and the flaky-CLI install/desync runbook (private-repo empty-hash root cause; `skillFolderHash` == git tree hash).
