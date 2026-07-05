# DISPATCH.md — delegation, model choice, escalation (verified 2026-07-05)

## §1 The commander does not descend
The main conversation is the scarcest resource in the session. Its job: decide, dispatch,
integrate conclusions, talk to the user. It does NOT: read big files end-to-end, sweep the
repo, fetch web/MCP content, or grind through mechanical edits. When in doubt whether to
dispatch, apply §2's numbers; if under every threshold, just do it inline — a subagent
spawn also costs real money and starts cold, so never dispatch what one Read/Grep answers.

## §2 Dispatch triggers (numbers, not vibes)
Dispatch a subagent when ANY of these holds; otherwise work inline:
- You expect to read >200 lines total to answer a question whose ANSWER you need, not the
  text itself (e.g. "where is dark mode handled?" → Explore. Editing a 300-line file you
  must modify anyway → inline, but read with offset/limit).
- Any WebFetch/WebSearch or content-heavy MCP pull (Notion pages, Gmail threads, Drive
  docs, PR file lists) → subagent fetches, distills, returns ≤30 lines.
- The same mechanical change across ≥3 files → one subagent applies all of them.
- The work is well-specified and >10 tool calls of grinding (bulk renames, batch checks).
- Verification of your own completed work → ALWAYS dispatched (§7), never inline.

## §3 Agent roster (available types verified in this harness)
| Type | Use for | Caveats |
|---|---|---|
| Explore | search/sweeps, "where/how does X" | **Does NOT load CLAUDE.md** — prompt must be self-contained. No Edit/Write, but HAS Bash: never ask it to mutate state. |
| Plan | designing an implementation approach | **Does NOT load CLAUDE.md** — include constraints in prompt. Same no-mutation caveat as Explore. |
| general-purpose | implementation, batch edits, research with writes | Loads CLAUDE.md. Full tools. |
| verifier (custom, .claude/agents/verifier.md) | acceptance checks per §7 | Read-only by design; report format fixed. |
| claude-code-guide | questions about Claude Code/SDK/API features | Answers from official docs; use before hard-coding harness facts. |
| claude, statusline-setup | default/fallback; statusline config | Rarely needed. |
Roster may differ in a future harness: the authoritative list is the "Available agent
types" system note at session start. If a type named here is missing, use general-purpose
plus the matching TEMPLATES.md prompt.

## §4 Model and effort — set them explicitly
Facts (confirmed against code.claude.com/docs/en/sub-agents.md, 2026-07-05):
- The Agent tool takes a per-call `model` override. Allowed aliases here: haiku, sonnet,
  opus (fable exists as an alias but assume it is NOT available to your session).
- There is NO per-call effort parameter. Effort comes from the agent definition's
  `effort:` frontmatter (low|medium|high|xhigh|max) or inherits the session default.
  To control effort, dispatch a custom agent that pins it (e.g. verifier pins high) or
  accept inheritance.
- Custom agents: `.claude/agents/<name>.md`, frontmatter supports name, description,
  tools, model, effort, maxTurns, etc. Explore/Plan skip CLAUDE.md; custom agents load it.

Tier guide:
- **haiku** — mechanical, fully-specified, low-blast-radius: bulk renames, format checks,
  file inventory, applying an already-worked-out pattern to N files. One strike (§6).
- **sonnet** — default worker for everything else: implementation, research, review.
- **opus** — escalation target; also first choice when the task is known-hard: cross-file
  refactors with tricky invariants, debugging with unclear cause, judgment-heavy review,
  second opinions.
Omitting `model` makes the subagent inherit the session model — fine when the session
model is already the right tier; specify explicitly when you want cheaper or stronger.

## §5 Report contract (put it in every dispatch prompt)
Every dispatch prompt contains three parts (templates: TEMPLATES.md):
1. **Goal + why** — what to produce and what decision it feeds, so the agent can cut scope
   sensibly when it hits surprises.
2. **Acceptance criteria** — checkable statements ("tsc shows no new errors vs baseline
   list pasted below", "every key in zh exists in en"), not adjectives ("clean", "solid").
3. **Report format** — "Return ≤30 lines: conclusions + file:line references. Any artifact
   longer than that (diffs, inventories, drafts) → write to a file (scratchpad for
   throwaway, repo for deliverables) and return the path. Never paste whole files back."
Also paste in whatever context the agent needs (relevant WORKLOG lines, baseline error
list, constraints) — subagents don't see your conversation, and Explore/Plan don't even
see CLAUDE.md.

## §6 Escalation / de-escalation ladder
- **haiku fails once** → don't retry haiku. Rewrite the prompt if the spec was bad, then
  send to sonnet.
- **sonnet (or session-tier) fails the same subtask twice** → stop. Write to WORKLOG.md:
  what was attempted, exact error/wrong output, your current hypothesis. Then dispatch
  opus WITH that full failure trace pasted in (an escalation without the trace just
  repeats the same mistakes at higher cost).
- **opus fails twice** → do not burn a third run. Go to JUDGMENT.md §4: usually the task
  is ill-posed — re-scope it or ask the user with your findings so far.
- **De-escalate**: the moment a stronger model has produced a working pattern (one correct
  example of the edit/output), batch the remaining instances to haiku/sonnet with the
  pattern pasted in as the spec.
- Hard cap, one arithmetic: max 2 attempts per tier (haiku: 1), max ONE escalation, so at
  most 4 attempts total on the same subtask (3 if it started on haiku). After that, do not
  dispatch again — change approach or ask the user (JUDGMENT.md §4). This is the same rule
  as CLAUDE.md rule 5: "failed twice at current tier ⇒ escalate", applied at most once.
- "Fails" means: acceptance criteria not met, or report is unusable (violates §5.3).

## §7 Verification is never self-verification
The author of a change — you, or a subagent — never signs off on it. When a unit of work
looks done:
1. Dispatch `verifier` (fresh context; if the roster lacks it, general-purpose + the
   VERIFY template in TEMPLATES.md).
2. Give it: what was supposed to change, the acceptance criteria, and how to check
   (files to read back, commands to run — e.g. `npx tsc --noEmit` diffed against the
   baseline in LETTER.md §1).
3. It returns PASS/FAIL per criterion with evidence (file:line, command output lines).
   FAIL → fix and re-verify; counts as a strike for §6.
For high-stakes judgment calls (user-visible zh-TW copy, irreversible actions, "which of
these designs"), add a second opinion: dispatch opus for an independent answer, or
generate 2–3 candidates and have a fresh agent pick the best against written criteria.

## §8 Never dispatch
- Talking to the user, deciding scope, precedence conflicts → main conversation only.
- Anything requiring credentials/outward actions (email, Notion writes, PR comments) →
  main conversation, and only on explicit user request (CLAUDE.md rule 6).
- git commit/push of the final result → main conversation, so you know what history holds.
