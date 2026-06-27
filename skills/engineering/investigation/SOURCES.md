# Sources

The grounding channels for an investigation pass. Each is a place to find *grounded* facts: query it with a `site:`-scoped search, then deep-fetch the authoritative hit. **Prefer primary sources** (official docs, changelogs, source repos) over the top SEO result — agents measurably drift toward content farms otherwise. Add sources freely; this catalog is meant to grow.

## Default tiers

| Tier | Source | Query it with | Best for |
|------|--------|---------------|----------|
| **Code & impl** | GitHub code / repos / issues | `gh search code "<symbol>"`, `gh search repos "<topic>"`, `site:github.com <topic>` | how people actually build it; real API shapes; prior art |
| | Stack Overflow | `site:stackoverflow.com <error / topic>` | concrete errors, gotchas, idioms |
| | Sourcegraph | `https://sourcegraph.com/search?q=<symbol>` | cross-repo usage patterns at scale |
| **Industry pulse** | TLDR | `site:tldr.tech <topic>` | what the field flagged recently — launches, deprecations |
| | Hacker News | `site:news.ycombinator.com <topic>`, `https://hn.algolia.com/?q=<topic>` | practitioner debate, failure stories, sentiment |
| | Lobsters | `site:lobste.rs <topic>` | higher-signal engineering discussion |
| | dev.to / changelog.com | `site:dev.to <topic>`, `site:changelog.com <topic>` | tutorials, release commentary |
| **Standards & docs** | Official docs | `site:docs.<vendor>.com <topic>`, `<tool> docs <topic>` | canonical current behavior — the ground truth |
| | MDN | `site:developer.mozilla.org <topic>` | web-platform standards |
| | RFCs / specs | `site:rfc-editor.org <topic>`, `<topic> specification` | protocol / format ground truth |
| | awesome-* lists | `site:github.com awesome <topic>` | curated landscape of the options |

## Opt-in tier — novel / algorithmic topics only

| Tier | Source | Query it with | Best for |
|------|--------|---------------|----------|
| **Research depth** | arXiv | `site:arxiv.org <topic>` | novel algorithms, primary research |
| | Google Scholar | `https://scholar.google.com/scholar?q=<topic>` | citations, prior art |
| | Papers with Code | `site:paperswithcode.com <topic>` | algorithm ↔ implementation |

Skip this tier for routine engineering — it's heavier and rarely changes a build decision.

## Search gotchas — these silently un-ground a brief

- **A `site:` miss is inconclusive, not absence.** `site:` doesn't rank and may omit indexed URLs — on an empty result, retry the query unscoped before concluding "no docs exist."
- **WebFetch returns cross-host redirects instead of following them** (e.g. `docs.anthropic.com` → `platform.claude.com`). Re-issue WebFetch on the returned target, or the source never loads and the brief gets written ungrounded.
- **GitHub code search indexes only the default branch** and needs sign-in for all-public-repo search — a "not found" can be an access / index artifact, not proof the symbol is unused.
- **Date-check fast-moving facts.** "Is X still current," answered from stale pretrained memory, is the most common silent failure — surface the source's publish date in the brief.

## Adding a source

A good source is *primary* (the project / vendor / author, not a recap) and *queryable* (a stable `site:` host or a CLI / URL search). Add a row to the right tier with its query recipe. Keep the catalog primary-first.
