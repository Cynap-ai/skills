---
"mattpocock-skills": patch
---

Recalibrate the **`investigation`** skill's auto-fire gate against an eval over 328 real session openers. The gate now leads with one deciding test — "do you need a vendor's DOCS or your own LOGS?" — and fires generously on third-party library/API/SDK/framework, version-currency, and approach-selection work (even when the vendor name is familiar but its current API isn't), while a dominant hard-skip keeps it off internal runtime debugging (reconciler/lifecycle/handler/deploy/alarm — ground truth is CloudWatch + logs + the codebase) and pure internal authoring. On the corpus this moved misses 9→0 and over-fires 18→9 versus the prior gate — a strict improvement on both axes.
