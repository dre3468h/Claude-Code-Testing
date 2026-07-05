# DIAGNOSIS.md — why the rules exist (written 2026-07-05 by a Fable-5 session)

This file explains the three biggest failure modes of this harness, each with the concrete
fix now encoded in the other .claude/ files. Read this when a rule seems arbitrary or when
you are tempted to skip one. Evidence for every claim was gathered and verified on
2026-07-05 in this repo.

## Failure #1 — Docs that lie: dangling references and a fictional repo map
**(biggest cause of wasted tokens AND lost trust in all instructions)**

What was found on 2026-07-05:
- The previous CLAUDE.md routed to seven files (`.claude/DISPATCH.md`, `JUDGMENT.md`,
  `TEMPLATES.md`, `MAINTENANCE.md`, `DIAGNOSIS.md`, `LETTER.md`, `WORKLOG.template.md`)
  — **none of which existed**. The `.claude/` directory itself did not exist.
- It described the repo as having `translations.ts`, `services/geminiService.ts`, a Vite
  config, and "no lockfile". Reality: none of those files exist, and `package-lock.json`
  does exist. `App.tsx` imports 10 local modules (`./types`, `./translations`,
  `./components/*`) that are all missing from the repo.

Why this is the #1 problem: a session that follows the routing table burns its opening
turns trying to read files that aren't there, then (worse) generalizes "the docs are
unreliable" and starts ignoring the rules that ARE correct. A weaker model is especially
vulnerable: it either hallucinates the missing file's contents or silently drops the rule.

Fix, now in force:
- Every path or command written into CLAUDE.md or .claude/*.md must be **verified in the
  same session that writes it** (file: `ls` it; command: run it; tool name: check the tool
  list). MAINTENANCE.md §3 makes this a mandatory post-edit check.
- CLAUDE.md carries a "facts verified on <date>" stamp. If the stamp is older than the last
  commit touching the repo's structure, re-verify before relying on it.
- The true, verified state of the repo (including what is MISSING) lives in LETTER.md §1.

## Failure #2 — Main-conversation token hemorrhage
**(biggest cause of context exhaustion and post-compaction derailment)**

The mechanics in this harness:
- The container is ephemeral and the conversation gets compacted when long. Anything not
  written to a file may be silently summarized away mid-task.
- ~100 MCP tools (github, Gmail, Drive, Notion) return full payloads into the main
  conversation unless told otherwise; `App.tsx` alone is 530 lines; `npm install` and
  similar commands print pages of noise.
- A subagent that pastes its whole exploration back doubles the cost: tokens spent in the
  subagent AND again in the parent.

Fix, now in force (numeric thresholds, not vibes):
- Reading >200 lines total for a question you only need a conclusion from ⇒ dispatch an
  Explore/general-purpose agent instead (DISPATCH.md §2).
- Any command expected to print >100 lines ⇒ `>file 2>&1` then read the tail.
- Subagent report contract: ≤30 lines, conclusions + `file:line` refs only; long artifacts
  go to files, return the path (DISPATCH.md §5, TEMPLATES.md).
- GitHub MCP calls: always paginate (5–10 items) and use `minimal_output: true` when the
  tool supports it.
- Any task with ≥3 steps ⇒ create `.claude/WORKLOG.md` from the template BEFORE step 1.
  After any compaction, read WORKLOG.md before doing anything else. (It is gitignored —
  session-local scratch state, not history.)

## Failure #3 — No verification baseline + author-verified "done"
**(biggest cause of false completion claims and misdirected fixes)**

What was found on 2026-07-05:
- There is no test suite, `npm run build` does not typecheck, and the baseline is broken:
  `npx tsc --noEmit` fails with 12 errors (10 missing-module TS2307 + 2 implicit-any
  TS7006 in App.tsx) and the app cannot build at all (no index.html, no vite.config, no
  entry point). Exact list: LETTER.md §1.

Two traps this sets for a weaker model:
1. It runs the gate, sees 12 pre-existing errors, assumes it broke something, and starts
   "fixing" files it never touched — burning the session on a rabbit hole.
2. Since the gate can never be green, it quietly stops running the gate at all and claims
   "done" on eyeball inspection.

Fix, now in force:
- "Done" for code = **no NEW errors versus the recorded baseline**, not zero errors.
  Procedure and the baseline itself: JUDGMENT.md §2 + LETTER.md §1.
- Acceptance is never run by the author of the change. Dispatch the `verifier` agent
  (defined in `.claude/agents/verifier.md`) with fresh context (DISPATCH.md §7).
- File deliverables are verified by read-back: re-read the written file and check it
  against the acceptance list, in the verifier's context, not the author's.

## Honest limits of these fixes
Decomposition, thresholds, and independent verification recover most *execution* quality
on a smaller model. They do NOT recover taste and open-ended judgment: ambiguous product
copy, tone decisions in zh-TW marketing text, "is this design good", novel trade-offs.
For those, JUDGMENT.md §6 gives the escape hatches: ask the user, get a second opinion
from a stronger dispatched model, or say plainly that the judgment exceeds the session's
reliable ability. Never fake confidence on a taste call.
