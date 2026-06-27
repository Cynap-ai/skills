---
"mattpocock-skills": minor
---

Add the **`investigation`** skill — model-invoked pre-task grounding. Before acting on an unfamiliar task it runs a lean, single-context web pass (WebSearch/WebFetch + `site:`-scoped lookups of official docs, GitHub, and industry sources like TLDR and Hacker News), seeded by first framing the task in the current codebase, and emits a short grounded brief: industry standard, best-in-class patterns, tips, recommendations for the task, and what to research next. It anchors on the leading word _ground_, encodes a hard skip gate (a one-sentence diff — typo, rename, log line — never fires) so it doesn't over-fire, and caps output at a ~1–2k-token brief with one source URL per claim. `SOURCES.md` is an extensible source catalog; `DEEP-WORKFLOW.md` carries an on-demand dynamic multi-agent escalation (fan-out across angles × sources, adversarially cross-verified) for broad/high-value topics, with `deep-research` named as the heavier escalation beyond it.

Also reconcile pre-existing drift in `skills/engineering/README.md`, which was missing `improve-harness`, `review`, and `resolving-merge-conflicts`.
