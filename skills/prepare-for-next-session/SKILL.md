---
name: prepare-for-next-session
description: This skill should be used at the end of a work session when the user says to "wrap up", "prepare .claude for next time", "save state for next session", "end of session", or wants the workspace left ready to resume. Captures only durable, non-obvious state into memory, turns the session's mistakes into fixes/lessons, and prunes the rest.
version: 0.2.0
---

# Prepare for Next Session

Leave the smallest set of notes that lets the next session resume without
re-deriving context, AND turn this session's mistakes into changes that stop them
recurring. Default to writing nothing; record only what clears the bar.

## Keep a fact or lesson only if it passes all three
- **Durable** — still true / still applies next session, not a detail of today's chat.
- **Non-obvious** — not recoverable from the repo, git history, or CLAUDE.md.
- **Actionable** — changes what the next session does or knows.

Fail any one, skip it.

## Steps
1. Scan the session for: unfinished work + the single next step; decisions with
   lasting effect; new user constraints/preferences; and **mistakes, corrections,
   and rework** — wrong turns you took, things the user had to correct, fixes that
   needed redoing, or a tool/skill/instruction that behaved unexpectedly.
2. For each kept fact OR durable lesson, update the **existing** memory file it
   belongs to; create a new one only if none fits. A lesson about how to work is a
   `feedback` memory (state the **Why** and the **How to apply**). One fact per
   file, never duplicated across files.
3. **Fix the cause, don't just log it.** When a mistake's root cause is a flaw in
   a skill, instruction, or config (not a one-off slip), make the concrete change
   there now — edit the skill / settings — so it cannot recur, and say what you
   changed. A memory note is the fallback only when the cause can't be fixed in
   place.
4. Delete memories that today proved wrong or obsolete.
5. Ensure `MEMORY.md` has one short pointer line per memory file — no content,
   no duplicates.
6. If the repo has a `STATUS.md` (or equivalent), keep "where we left off / next
   step" there, not in memory — don't store it in both places.
7. Report in 2-3 lines what was added / updated / pruned, plus any lesson recorded
   or skill/config changed — or that nothing met the bar.

## Avoid
- Re-summarizing what code or git already shows.
- Creating a session-log or changelog file.
- Two memories for one fact, or restating something already in MEMORY/STATUS.
- Recording a one-off slip or a blow-by-blow of the session — keep only lessons
  that will change future behavior.
