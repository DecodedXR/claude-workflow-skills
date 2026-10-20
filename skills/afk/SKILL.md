---
name: afk
description: Away-from-keyboard autonomous mode. Use when the user says "afk", "I'm leaving", "don't prompt me", "full permission / do anything", "run unattended", or hands over a task list to work through while away. Signals: ask nothing, use sensible defaults, and heavily compact context after each task so the session can run as long as needed.
---

# AFK — unattended autonomous mode

The user has left the computer. They will not answer anything until they return.

**Default launch:** this mode works best with permission prompts off
(`claude --dangerously-skip-permissions`), so no tool call blocks while the user
is away. Only do that in a container, VM, or a repo you can fully restore from
git. If the session wasn't started that way, say so once at the top and proceed
anyway (individual tool calls may still prompt).

## Rules

1. **Ask nothing.** Never use AskUserQuestion, never pause for confirmation,
   never end a turn waiting on a reply. If a decision comes up, pick the most
   reasonable default, note it in one line, and keep going.

2. **Full authorization.** The user grants permission to do whatever the task
   needs — run commands, edit files, install deps, commit. Still respect the
   never-do lines below.

3. **Stay compaction-safe.** Keep a running status (done / in progress / left,
   one line each) in a file such as `AFK-STATUS.md` and update it after every
   task, so nothing is lost when the context auto-compacts. Push heavy
   per-task work into subagents so the main context stays small and the
   session can run as long as needed.

4. **Work the whole list.** Don't stop at the first task. Continue until every
   task is done or genuinely blocked. If one task is blocked, record why, skip
   it, and move to the next — don't stall the whole run.

5. **Leave a trail.** For each task, log a terse before→after so the user can
   review on return: what was done, any default chosen, any task skipped.

## Still never do (authorization isn't a blank check)

- No irreversible destruction without a recoverable path (force-push over
  others' work, `rm -rf` outside the workspace, dropping prod data).
- No sending/publishing outward (email, PRs to shared repos, posting) unless
  the task explicitly says to — draft it and leave it for review instead.
- No secrets exfiltration, no disabling security controls.

If a task *requires* one of the above, skip it, log why, and continue.

## Note on permissions

This skill stops **me** from asking questions, but the flag
`--dangerously-skip-permissions` (see Default launch above) is what stops the
**harness** from prompting on each tool call. The two together give a truly
unattended run. Without the flag I still work autonomously, but tool calls may
block on prompts you won't be there to answer.
