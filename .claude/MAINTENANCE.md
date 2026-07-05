# MAINTENANCE.md — how to change the rules safely

## §1 Change authority
May change WITHOUT asking the user:
- Appending a lesson to LESSONS.md (§4) or a fact correction with evidence (e.g. a path
  in CLAUDE.md is stale because the repo changed — fix it, cite the verifying command in
  the commit message).
- Updating LETTER.md §1's verified-state snapshot after re-verifying it.
- Adding a new TEMPLATES.md template or a new custom agent under .claude/agents/.
Must ASK THE USER first (batched question, with recommendation):
- Changing/removing any Iron rule in CLAUDE.md, the precedence order, or the domain line.
- Loosening any threshold (retry caps, line limits, verification requirements).
- Changing zh-TW copy policy or anything that affects outward actions (rule 6).
- Deleting any .claude/*.md file or moving content between files (routing changes).

## §2 Before editing any existing CLAUDE.md / .claude/*.md
1. `cp <file> .claude/backups/<file-basename>.$(date +%F).bak` (backups/ is gitignored;
   git history is the durable backup — the .bak protects you only within this session).
2. Make the edit as a NEW commit touching only doc files, message prefixed `docs(claude):`.
3. Run the §3 check before committing.

## §3 Post-edit reference check (mandatory, mechanical)
After ANY edit to CLAUDE.md or .claude/*.md, run from repo root:
```
grep -rhoE '\.claude/[A-Za-z0-9._/-]+' CLAUDE.md .claude/*.md .claude/agents/*.md \
 | sed 's/[).,;:]*$//' | sort -u \
 | grep -v -e '\.claude/WORKLOG' -e '\.claude/backups' -e 'settings.local' \
 | while read -r p; do [ -e "$p" ] || echo "MISSING: $p"; done
```
Any MISSING line ⇒ fix the reference or create the file BEFORE committing. (WORKLOG.md and
backups/ are excluded because they are session-local by design.) Also spot-check that any
command you added actually runs, and any tool/agent name you added appears in the session's
tool list or agent roster. This check exists because the 2026-07-03 CLAUDE.md shipped seven
dangling references and a wrong repo map — see DIAGNOSIS.md failure #1.

## §4 LESSONS.md — the learning loop
Append an entry when: a rule conflict surfaced (CLAUDE.md rule 8), an escalation was
needed, a verification caught a real error, the user corrected you, or you wasted >20
tool calls on something a rule change would prevent.
Format (one entry, ≤6 lines):
```
## 2026-07-05 · short title
Trigger: what happened (1 line, concrete)
Cost: tokens/time/user trust impact (1 line)
Rule change: none | proposed | applied → <file §, commit>
```
A lesson that recurs 2+ times MUST graduate: propose the rule change (per §1 authority)
instead of logging it a third time.

## §5 Pruning — docs must not grow monotonically
Caps: CLAUDE.md ≤ 90 lines; each other .claude/*.md ≤ 160 lines; LESSONS.md ≤ 100 lines.
When over cap: merge duplicates, delete rules that never fired (check LESSONS.md), move
narrative into DIAGNOSIS.md or delete it. Pruning that drops any user-approved rule needs
user sign-off (§1). When LESSONS.md is pruned, keep the distilled rule changes, drop the
stories.

## §6 After any docs change
1. §3 reference check passes.
2. Dispatch verifier: read back the changed file(s); criteria = "renders as intended,
   no contradiction with CLAUDE.md precedence chain, no dangling refs".
3. Commit, push, and remind the user: .claude/ changes reach future sessions only after
   merge to main.
