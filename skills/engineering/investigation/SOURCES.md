# Sources

The grounding channels for an investigation pass. Each is a place to find *grounded* facts: query it with a `site:`-scoped search, then deep-fetch the authoritative hit. **Prefer primary sources** (official docs, changelogs, source repos) over the top SEO result — agents measurably drift toward content farms otherwise. Add sources freely; this catalog is meant to grow.

**Lean pass uses the marked default trio only.** The lean grounding pass has a ~4-call budget (one broad → one `site:` → one deep-fetch → one narrowing follow-up), so it draws on the three **[lean-default]** rows below — everything else is **[deep-only]**, reached when deep mode fans out.

## Default tiers

| Tier | Source | Query it with | Best for |
|------|--------|---------------|----------|
| **Standards & docs** | **`llms.txt` / `llms-full.txt`** **[lean-default]** | Try `https://<docs-host>/llms.txt` (index) or `/llms-full.txt` (full) **FIRST** for any vendor with dev docs | a machine-readable, agent-targeted docs map — beats guessing doc URLs or landing on SEO recaps |
| | Official docs | `site:docs.<vendor>.com <topic>`, `<tool> docs <topic>` | canonical current behavior — the ground truth |
| | MDN | `site:developer.mozilla.org <topic>` | web-platform standards |
| | RFCs / specs | `site:rfc-editor.org <topic>`, `<topic> specification` | protocol / format ground truth |
| | awesome-* lists **[deep-only]** | `site:github.com awesome <topic>` | curated landscape of the options |
| **Code & impl** | GitHub code / repos / issues **[lean-default]** | `gh search code "<symbol>"`, `gh search repos "<topic>"`, `site:github.com <topic>` | how people actually build it; real API shapes; prior art |
| | Stack Overflow **[deep-only]** | `site:stackoverflow.com <error / topic>` | concrete errors, gotchas, idioms |
| | Sourcegraph **[deep-only]** | `https://sourcegraph.com/search?q=<symbol>` | cross-repo usage patterns at scale |
| **Industry pulse** | TLDR / Hacker News **[lean-default: pick one]** | `site:tldr.tech <topic>`, `site:news.ycombinator.com <topic>`, `https://hn.algolia.com/?q=<topic>` | what the field flagged recently — launches, deprecations, sentiment |
| | Lobsters / dev.to / changelog.com **[deep-only]** | `site:lobste.rs <topic>`, `site:dev.to <topic>`, `site:changelog.com <topic>` | higher-signal discussion, tutorials, release commentary |

## Opt-in tier — novel / algorithmic topics only (deep mode)

| Tier | Source | Query it with | Best for |
|------|--------|---------------|----------|
| **Research depth** | arXiv | `site:arxiv.org <topic>` | novel algorithms, primary research |
| | Google Scholar | `https://scholar.google.com/scholar?q=<topic>` | citations, prior art |
| | Papers with Code | `site:paperswithcode.com <topic>` | algorithm ↔ implementation |

Skip this tier for routine engineering — it's heavier and rarely changes a build decision.

## Search gotchas — these silently un-ground a brief

- **A `site:` miss is inconclusive, not absence.** `site:` doesn't rank and may omit indexed URLs — on an empty result, retry the query unscoped before concluding "no docs exist."
- **WebFetch and redirects.** It *returns* a **cross-host** redirect's new URL instead of following it (e.g. `docs.anthropic.com` → `platform.claude.com`) — re-issue WebFetch on the target. A **same-host** trailing-slash/301 can be wrongly rejected ("Redirect not allowed") — retry **with** the trailing slash. And WebFetch can't render JS-only pages — a near-empty SPA-docs fetch is a tooling limit, not missing content (fall back to the doc's `llms.txt` or a `site:` search).
- **GitHub code search has hard caps.** Default branch ONLY (use `branch:` otherwise); login required (a logged-out not-found is an *access* artifact); only files **<384 KB** and the **first 500 KB** of a file are indexed; forks are indexed only if they out-star the parent. A no-results is **never** proof a symbol is unused.
- **WebSearch's "US-only" note is a false geo-restriction.** The tool self-describes as US-only, but it works fine outside the US — never let that note deter or deprioritize a grounding search. If a first search returns nothing, the US note is *never* the reason; reformulate the query.
- **Date-check fast-moving facts.** "Is X still current," answered from stale pretrained memory, is the most common silent failure — surface the source's publish date in the brief, and prefer a `changelog`/`releases` page over a tutorial.

## Adding a source

A good source is *primary* (the project / vendor / author, not a recap) and *queryable* (a stable `site:` host or a CLI / URL search). Add a row to the right tier with its query recipe, and tag it **[lean-default]** or **[deep-only]**. Keep the catalog primary-first.
