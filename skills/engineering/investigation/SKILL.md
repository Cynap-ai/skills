---
name: investigation
description: Ground an unfamiliar task in real external facts before acting — a short web-grounded brief (industry standard, current best-in-class approach, tips, recommendations, what to research next) built from WebSearch/WebFetch + `site:`-scoped lookups of official docs, GitHub, and industry sources (TLDR, Hacker News, …), seeded by first framing the task in this codebase. Use when about to work with an unfamiliar library, API, framework, integration, or fast-moving tool, when choosing an approach where the current standard matters, or when unsure whether "X is still the right way." Skip when the change is a one-sentence diff you could already describe (typo, rename, log line) or the area is already familiar. Escalate to deep mode for broad/high-value topics, and to the `deep-research` skill when every claim must be cited into a standalone report.
---

# Investigation

Ground an unfamiliar task in real external facts **before** you act. The output is a short **grounded brief** — enough to start correctly — not a report.

## When to ground

Ground when the task touches an **unfamiliar** library, API, framework, integration, or fast-moving tool; when you're choosing an approach where the current industry standard matters; or when you're unsure "is X still the right way." **Skip** a one-sentence diff you could already describe (typo, rename, log line) or an already-familiar area — grounding the known is a no-op tax, and over-firing is this skill's real failure mode.

## The grounding pass — single-context, bounded

Stay in one context: a handful of targeted queries, then the brief. **Never fan out to subagents and never read broadly** — if grounding needs that, it's *deep mode*, not this.

1. **Frame first (internal).** Before searching, ground in your own premise — what the task actually needs, what already exists in this codebase/architecture, the real goal and its constraints. Read the relevant code/docs/issue; a claim about the codebase cites a real `file:line`, never a guess. The frame is what makes the next step's queries *yours*, not generic.
2. **Research (external), seeded by the frame.** Search shaped by the real stack the frame named. Default move: one broad orienting query → `site:`-scoped lookup of the authoritative source → single deep-fetch of the top hit → at most one narrowing follow-up. Prefer primary sources (official docs/changelogs, the source repo, GitHub code search) over the top SEO hit. [`SOURCES.md`](./SOURCES.md) holds the per-source query recipes (GitHub, TLDR, Hacker News, docs, …) and the search gotchas that silently un-ground a brief.
3. **Write the brief** — a few hundred words (~1–2k tokens), sections: `TL;DR · Problem frame · Industry standard · Elevation (best-in-class) · Tips & gotchas · Recommendations for THIS task · What to research next · Sources`. **One source URL per claim — drop any line you can't ground, and say what you couldn't confirm.** Date-check anything fast-moving and surface the source's date.

Save the brief to `docs/investigations/YYYY-MM-DD-<slug>.md`, post it, then hand off to `/grill-with-docs` — its open questions and recommendations are the grilling's ammunition.

## Deep mode — on demand, not the default

When the task is genuinely broad or high-value (many sources, several angles, more than one context), escalate to a dynamic multi-agent run: agents fan out across angles × sources, adversarially cross-verify, then synthesize. It is the **only** path that fans out — fan-out costs ~15× a single pass, so reach for it deliberately. The workflow-script template and guardrails live in [`DEEP-WORKFLOW.md`](./DEEP-WORKFLOW.md). Reach for it on `/investigation deep <topic>`, or when you judge one lean pass won't cover the task.

When even that won't do — every claim cited and cross-checked into a standalone report — escalate to the `deep-research` skill.

## Done when

The brief exists, every claim carries a source (or is flagged unconfirmed), and it's saved and handed off. A brief padded past ~2k tokens, or carrying an ungrounded assertion, isn't done — it's drifting toward deep-research; cut it back or escalate deliberately.
