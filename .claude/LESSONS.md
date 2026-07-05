# LESSONS.md — append per MAINTENANCE.md §4; prune at 100 lines (keep rule changes, drop stories)

## 2026-07-05 · Docs shipped with dangling references
Trigger: CLAUDE.md (2026-07-03) routed to seven .claude/ files that did not exist and
described repo files (translations.ts, geminiService.ts) that were never uploaded.
Cost: every session's opening turns wasted; instructions lost credibility wholesale.
Rule change: applied → MAINTENANCE.md §3 mandatory reference check; CLAUDE.md facts
stamped with verification date.

## 2026-07-05 · Baseline is broken; "green gate" impossible
Trigger: `npx tsc --noEmit` fails with 12 pre-existing errors (repo incomplete);
build/dev cannot run at all.
Cost: risk of false "I broke it" rabbit holes or silently skipped gates.
Rule change: applied → "done = no NEW errors vs LETTER.md §1 baseline" (CLAUDE.md rule 4,
JUDGMENT.md §2).

## 2026-07-05 · Adversarial review caught 8 defects in the freshly written manual
Trigger: fresh-context review of .claude/ docs found an intra-file retry-cap
contradiction (DISPATCH §6), an unexecutable "EXACTLY this output" baseline claim
(LETTER §1), unverifiable verifier criteria (MAINTENANCE §6), and 5 more.
Cost: one review dispatch; without it these would have corrupted the core verify loop.
Rule change: applied → fixes in commit tagged docs(claude); keep mandatory fresh-context
review for any rewrite touching ≥3 institution files.
