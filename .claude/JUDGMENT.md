# JUDGMENT.md — decision rubrics (each rule: signal → action, with one good call ✓ and one bad call ✗)

## §1 When to escalate to a stronger model
Escalate when the *shape* of the difficulty is reasoning, not effort. Signals:
- Your second attempt failed for a DIFFERENT reason than the first (you're exploring a
  space you don't understand, not converging on a fix).
- You cannot state, in one sentence each, (a) the root cause and (b) why your next attempt
  will work. If either sentence is missing, more retries at this tier are dice rolls.
- The fix keeps growing: you came to change one file and are now touching four.
✓ Good: "tsc error persists after two fixes; my two fixes contradicted each other" →
  write trace to WORKLOG, dispatch opus with the trace (DISPATCH.md §6).
✗ Bad: "npm install failed with ETIMEDOUT" → escalating model. Network errors, missing
  env, permission denials are ENVIRONMENT problems — no model tier fixes them; retry with
  backoff or report to user.

## §2 When is it actually done
"Done" requires ALL of:
1. Every acceptance criterion checked with evidence (command output or file:line), by a
   fresh verifier agent — not by whoever authored the change (DISPATCH.md §7).
2. Code: `npx tsc --noEmit` shows no errors beyond the LETTER.md §1 baseline. If your
   change plausibly affects runtime behavior and the app can run, exercise it; today the
   app cannot run (LETTER.md §1), so say so explicitly in your completion claim.
3. File deliverables: read back the final on-disk/pushed version and check it against the
   request point by point (nothing dropped, no placeholder like "TODO"/"..." left).
4. Work is committed AND pushed (`git log origin/<branch>` shows it).
5. The completion message states what was verified and what was NOT verifiable.
✓ Good: "Done: en/zh key parity restored; verifier confirmed 0 new tsc errors and key-set
  equality (report: PASS 3/3); pushed as abc1234. Not verified: visual rendering — app
  can't build."
✗ Bad: "I've updated the file and it should work now." ("should" = not done; nothing was
  verified; not pushed = doesn't exist, CLAUDE.md rule 1.)

## §3 When to stop and ask the user
Ask (batched, with your recommendation) when the decision is irreversible, outward-facing,
or changes something the user explicitly approved. Concretely:
- Changing meaning/tone of zh-TW user-facing copy (tone is user-approved).
- Anything leaving the repo: emails, Notion/Drive edits, PR comments, publishing.
- Deleting or overwriting work you didn't create, force-pushes, history rewrites.
- Two contradictory instructions where precedence (CLAUDE.md rule 8) doesn't settle it.
- After the escalation ladder is exhausted (DISPATCH.md §6 cap).
Do NOT ask when the answer is discoverable in the repo/docs, when it's a reversible
implementation detail, or to seek reassurance ("shall I proceed?").
✓ Good: "The CTA reads 論文代寫服務; making it compliant-sounding changes its meaning.
  Options: A (keep), B (soften, draft attached). I recommend B. Which?"
✗ Bad: asking "should I use a <button> or an <a> tag?" — reversible detail, decide and note it.
✗ Also bad: silently "improving" the zh copy tone because it seemed unprofessional.

## §4 Signals you're on the wrong path (change approach — retrying is harmful)
- Each fix spawns a new error in a DIFFERENT place (you're fighting the design).
- You are about to edit generated/vendored/lock files or add a hack you'd hide in review
  (`@ts-ignore`, `any`, `setTimeout` to dodge a race) to make a gate pass.
- The diff for a "small" task exceeds ~150 lines or 5 files.
- You're undoing your own earlier edit from this same session (loop detected).
- The plan requires a fact you've now verified is false (e.g. "wire the existing
  geminiService" — it doesn't exist).
Action: stop mid-attempt, write state to WORKLOG.md, re-derive the approach from the
actual error/goal (or dispatch Plan with self-contained context), and if the task itself
was mis-specified, tell the user what you found instead of delivering the wrong thing.
✓ Good: "Task says fix the header component; repo has no components/ dir at all. Reporting
  repo-incomplete instead of inventing a Header.tsx."
✗ Bad: third retry of the same edit with small wording changes, hoping tsc changes its mind.

## §5 Quality floor (checks a weak model can run mechanically)
Before handing anything to the verifier:
- Types: `npx tsc --noEmit` diffed against baseline (LETTER.md §1).
- Translations (when translations.ts exists): zh and en key sets identical — check with a
  script/grep, not by eye; report the diff of keys if any.
- No secrets: `git diff --cached` contains no API keys, tokens, or .env contents.
- No placeholders left: grep your changed files for `TODO`, `FIXME`, `...`, `省略`.
- Copy changes: zh text shown to the user for approval BEFORE commit if meaning/tone moved (§3).
- Every new/changed .claude/*.md reference: run MAINTENANCE.md §3's reference check.

## §6 Honest limits — what these rubrics cannot recover
Checklists recover execution quality. They do NOT make a smaller model good at: taste
(is this copy persuasive? is this layout elegant?), ambiguous product intent, novel
trade-offs with no written criterion. When a task is mostly that kind of judgment:
1. Say so out loud in the conversation ("this is a taste call, my confidence is low").
2. Prefer: generate 2–3 genuinely different candidates → fresh agent (opus) ranks them
   against written criteria → user picks. Ranking against criteria is more reliable for a
   weak model than creating the single best answer.
3. Or ask the user for the criterion you're missing (one question, with options).
4. Never bluff a confident aesthetic verdict. "B is safest given X; a stronger model or
   your call would beat my judgment here" is an acceptable, correct answer.
