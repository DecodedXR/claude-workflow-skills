---
name: autopilot
description: Autonomously pick the single most important tracked task that can be FINISHED with zero human intervention, implement it test-first, prove it with an independent adversarial verifier, and land it via a PR that auto-merges ONLY if CI passes — no checkpoints, one task per run. Use ONLY when the user explicitly asks to autonomously do and ship the next thing: "autopilot", "do the next unblocked task", "knock out the next task without me", "advance the project on your own". Invoking it IS the standing authorization to open and auto-merge the PR. It never auto-SENDS anything (drafts only), never works on anything tied to an unresolved blocker that needs information or a decision it cannot obtain on its own, and BAILS (leaving the PR open or the tree clean) rather than guessing or forcing a green. Pass `ultra [+Nk]` for a budget-bounded multi-agent deep mode (an adversarial verifier panel + a dimensioned review) that scales depth toward a per-run token target but NEVER pads a task that does not need it.
version: 1.0.0
---

# Autopilot — Plan the Next Unblocked Task and Ship It

A deliberately-invoked, zero-touch mode: choose the one tracked task that can be
carried to landed-on-the-default-branch **without a human decision**, build it
test-first, **prove it with verification you did not author**, and land it through a
PR that **auto-merges only if CI is green**. One task per run.

> Project-agnostic by design. If a repo ships its own project-level `autopilot`
> (`<repo>/.claude/skills/autopilot/`), that one overrides this and should be
> preferred there — it encodes that project's task sources, test command, and
> invariants. This generic version adapts to whatever the current repo provides.

The danger this skill is built around: its pre-land safety net is "tests green +
review," but autopilot writes the test, the code, *and* the review — one blind spot
shared three ways. So the green is never trusted on its own: an **independent
adversarial verifier** and **CI** must also pass, and on any doubt it leaves the
work as an open PR for you rather than merging.

## Hard off-limits — never select, never touch

**Do not work on anything tied to an unresolved blocker that needs information or a
decision you cannot obtain autonomously** — full stop, not even a draft. That
includes anything depending on an unconfirmed external fact (a value, a spec, a
third-party answer), an unmade product/API judgment that is rightfully the user's,
or a hardware/physics/safety quantity you'd have to guess. If completing a task
would require guessing one of these, it is off-limits — never invent a value to keep
a task alive. Domain-critical code (auth, payments, migrations, safety/interlocks,
anything that can corrupt data or re-energize an output) is **not** carved out — it
is fair game, but guarded by the verifier + CI gate below, not by avoidance.

## Eligibility (besides the off-limits rule)

A task is eligible only if it can be **finished and proven green** with zero human
intervention. Skip it if it can't be verified by the existing test suite without
real hardware or manual steps, or if it's blocked/deferred. A task that hinges on a
product/API judgment you'd want to own is allowed to be *built*, but it must **not
auto-merge** — open the PR and leave it for you (see the merge gate). If no eligible
task exists, do **not** invent busywork: report the top blocked need, offer to draft
an escalation, and stop.

## 0. Pre-flight integrity gates (bail if any fails)

Refuse to start unless **all** hold — you cannot tell a regression from a
pre-existing fault on a dirty or red baseline:
- Working tree is **clean** (no uncommitted user work to commingle or clobber).
- The full local suite is **already green** (discover and run the project's test
  command — `package.json` scripts, `Makefile`, `pytest`, `cargo test`, CI config,
  etc.; if you cannot determine it, bail and ask).
- The default branch is up to date (`git fetch origin`) and its **latest CI run is
  green** (`gh run list --branch <default> -L1` when a GitHub remote exists) — don't
  pile onto an already-broken baseline.
- You were invoked **interactively**, not from `/loop` or `/schedule`. Autopilot
  must never run unattended on a timer.

## 1. Select (read the task sources, rank by risk-first)

Read whatever this repo uses to track work — in priority order as available: an
explicit status/roadmap doc (e.g. `STATUS.md`, `TODO.md`, a "next session" note), a
review/issue ledger (open = actionable, deferred = skip), the issue tracker (`gh
issue list`), and any project memory/CLAUDE.md notes. Rank **risk-first** (anything
touching security, data integrity, money, or a safety/availability path outranks
cosmetic work), then known-root-cause over open-ended, then severity. Prefer a
**bounded** item — a small diff is easier to verify and to revert. Pick the single
top eligible task.

## 2. Verify it is still real

Tracking prose lags the code by a session. Re-read the cited code **as it is now**
and `git log`/grep to confirm no later commit already fixed it. If already done, say
so and re-pick.

## 3. Plan, then implement — test-first, and prove the test isn't a tautology

**Plan before you edit, and surface it.** Before the first edit, lay out the design
explicitly in your reply: the root cause; the chosen fix at the right altitude and
the alternatives you rejected; the exact files/functions to change; the RED-test
design; and the invariants/contracts that must be preserved. For a non-trivial pick,
reach for the `Plan` agent (or a `plan-next-task` skill if the repo has one) first.
This stays **zero-touch** — *show* the plan, do **not** pause for approval (a hard
approval checkpoint contradicts this skill's autonomous design).

Then:
- Write the **RED repro first**; run it, confirm it fails **for the right reason**.
- Implement the minimal correct fix. Handle failure paths fully — error returns that
  don't raise, thrown exceptions, and abort/cancellation. Centralize any
  guard/constant used at ≥2 sites. Every docstring/contract promise becomes a test.
- **Prove the test is non-tautological:** with the fix reverted the new test must go
  RED again, and green only with the fix in place — otherwise it proves nothing.
- Keep the project's invariants (its layering, locking, validation boundaries,
  conventions). Verify any external/wire/hardware claim against the authoritative
  source, never against green tests.
- Run the **full local suite** to green — including any slow/excluded tiers CI
  skips, so you don't miss a regression CI won't catch. **Never weaken, skip, or
  delete a test, and never `--no-verify`, to reach green.** If it won't converge,
  STOP per §8.

## 4. Independent adversarial verification (the green you didn't author)

Spawn a **separate verifier agent in an isolated git worktree** (mutating verify
agents corrupt a shared tree — isolate them) that did **not** see your
implementation reasoning, prompted to **refute** that the change is correct and safe
and to find an input that breaks it — not to bless it. For any contract the change
touches, the verifier must exercise it with **real fault injection over the actual
stack** (the project's existing fakes/mocks/fixtures), never freshly-authored stubs.
If it finds a hole, fix it and re-verify, or bail per §8. This independent pass is
the gate that the author-written green cannot be.

## 5. Review (local)

Run `/code-review` (high or max) over the diff — **local, not a billed cloud
review** unless the user opted in. Include an invariant/contract-audit angle and, at
a phase boundary, composition/interleaving scope. Apply confirmed fixes and
re-green. A finding that needs a **judgment call** (not a clear fix) is not
auto-mergeable — carry it to the PR and leave it for you.

## 6. Land via PR — auto-merge ONLY if CI is green

Record the pre-change HEAD first (for the rollback line in your report). Then:
- Branch off the default branch (`fix/...`, `feat/...`); **stage only the files this
  task touched** (never `git add -A`); `git commit -F <file>` (write the message to a
  file rather than fighting shell quoting) with a `Co-Authored-By` footer and an
  `Autopilot: <task ref>` trailer so every autonomous change is greppable.
- Fold any tracking-doc update (status/decision-log entry, ledger/issue row flip
  referencing the PR # and the rollback HEAD) **into this same branch**, so it lands
  atomically and only reads as "landed" if the PR actually merges. This also avoids a
  post-merge direct-to-default-branch push, which an interactive auto-mode classifier
  may block.
- `git push origin <branch>` (**the branch, never the default branch**) and open a PR
  with `gh pr create`. The PR body records the task + why, the RED test, the
  verifier's verdict, the review outcome, and the rollback HEAD.

**Merge gate — auto-merge (`gh pr merge --merge --delete-branch`, a real merge
commit) ONLY IF every one of these is true:**
1. full local suite green, and the RED test proven non-tautological;
2. the independent adversarial verifier passed;
3. `/code-review` left no unresolved or judgment-call finding;
4. **CI is green** — watch it to conclusion (`gh pr checks <pr> --watch`);
5. you are genuinely confident the change is correct and safe.

If **any** fails — including plain doubt — **do not merge.** Leave the PR open and
report it for review. **Never fall back to a direct push to the default branch** if
the PR/CI path is unavailable (e.g. `gh` unauthenticated, branch protection needs a
human review): the CI gate is the point — bail and report instead.

## 7. After a successful merge only

The tracking-doc update already merged with the change (§6) — no post-merge
bookkeeping push needed. Then, in order:
1. If the repo has a status-update/email skill, save a **draft** reflecting what
   landed. **Never send.**
2. **As the final beat, run the `prepare-for-next-session` skill** (if present) to
   close out the run: capture only durable, non-obvious lessons as memories, turn any
   misstep into a fix, and prune stale memories. Don't duplicate what the tracking
   docs or git already record. Within an autopilot run, do **not** autonomously edit
   skill/config files (see Forbidden operations).

If the PR did not merge, skip all of this and just report the open PR.

## Forbidden operations (never, autonomously)

Weakening/skipping/deleting a test or using `--no-verify` to reach green;
force-push, squash-merge, or rewriting history; editing dependencies, CI config, git
hooks, settings, secrets, or the skill/memory infra; `git add -A` or committing
secrets/large binaries; running on a schedule/loop. Any of these means **stop and
ask**, not proceed.

## 8. Stop / bail

- **Done & merged:** report the task, the diff, the test + verifier + review + CI
  results, the merge commit, the rollback HEAD, and any saved draft.
- **Bail (clean):** if nothing is eligible; if the work surfaces an off-limits
  blocker or a decision that's yours; if tests won't go green; if the verifier finds
  a hole you can't close; or if CI/branch-protection blocks the merge. Revert or
  stash the partial work so the tree is clean, and report exactly where you stopped,
  why, and the decision needed. **Never merge a half-done or guessed change, and
  never force a green to get past the gate.**

## Avoid

- Guessing an external/hardware/physics/safety value, or touching an unresolved
  blocker, to keep a task alive.
- Trusting your own green: the adversarial verifier and CI must agree before merge.
- Auto-**sending** anything; auto-merging a judgment-call change; falling back to a
  direct push when the PR path is blocked.
- Inventing busywork when everything real is blocked — escalate instead.
- More than one task per invocation.

## Ultra mode — `autopilot ultra [+Nk]` (budget-bounded multi-agent depth)

Engaged ONLY when invoked as `autopilot ultra` (optionally with a `+Nk` token
directive). Everything above still holds — same selection, off-limits rules, **one
task per run**, and the **same merge gate, only stricter**. Ultra spends a token
surplus on *independent adversarial breadth* — the direct antidote to this skill's
core risk (it authors the test, the code, AND the review). Invoking the skill is
itself the `Workflow` authorization; use the `Workflow` tool for the fan-outs below.

**Budget contract — `budget` is SESSION-cumulative, ≠ your run quota.** The Workflow
`budget` global measures output tokens spent **this session so far** — NOT just this
run. So snapshot it at the start and bound the RUN on the delta. From the user's
`+Nk` directive the script derives:
- `const startSpent = budget.spent();` — taken once, first thing;
- `HARD = budget.total ?? <a sane per-run cap>` — per-run absolute ceiling;
- `SOFT = round(HARD * 2 / 3)` — preferred stopping point;
- gate EVERY loop on the per-run delta `spentThisRun = budget.spent() - startSpent`
  (`spentThisRun < SOFT`; hard-stop at `spentThisRun >= HARD`), so `+Nk` means "this
  RUN spends ≤ Nk" regardless of what the session already spent.

Report `spentThisRun` at the end so the user can re-tune `+Nk`. The run cannot read
the live account quota; it only honors the per-run token target you set.

**Never pad — this overrides the budget.** Stop the instant work is *productively*
exhausted, no matter how little was spent; autopilot never invents work to hit a
number. Every loop is **dry-detection first, budget second**:
- run a round; if it surfaced **nothing new and real**, increment a `dry` counter;
- stop when `dry >= 2` (two consecutive empty rounds) **OR** `spentThisRun >= SOFT`;
- continue past `SOFT` toward `HARD` **only** while rounds keep surfacing *real,
  confirmed* issues. A small/clean task finishes far under `SOFT`, and that is the
  correct outcome — under-spending is never a bug.

**§3 Plan (ultra) — judge panel, only if non-trivial.** For a multi-file or
non-obvious fix, `parallel()` N independent approach proposals from different angles,
score them, synthesize the winner and graft the best of the runners-up. SKIP for a
bounded single-spot fix — a panel there is padding.

**§4 Verify (ultra) — a perspective-diverse adversarial panel.** Replace the single
verifier with a `parallel()` of worktree-isolated refuters, each a **distinct lens**,
each prompted to *break* the change with fault injection over the project's real
stack, never author stubs:
- correctness / contract-faithfulness;
- concurrency & interleaving;
- failure-paths (error returns + exceptions + abort/cancellation);
- faithfulness to the authoritative external source;
- adversarial inputs / boundaries.

**Worktree caveat:** if `agent({isolation:'worktree'})` fails (e.g. on Windows with
long legacy paths, set `git config core.longpaths true` or prune), fall back to
**read-only parallel verifiers** (no isolation — safe to share the tree) and do the
empirical mutation / non-tautology checks yourself, serially, not in parallel on the
shared tree.

**Majority-refute ⇒ bail** (leave the PR open, per §8). While dry-detection still has
budget, add *fresh* lenses or re-verify a confirmed-sound finding from a new angle —
never re-run an identical lens just to spend tokens.

**§5 Review (ultra) — dimensioned find → verify → synthesize.** Replace the single
`/code-review` with a `pipeline()`: per dimension (correctness/bugs,
contracts/altitude, concurrency, conventions) find candidates, then **adversarially
verify each finding** (majority vote) before it counts; loop-until-dry; synthesize
and apply only confirmed fixes, then re-green. A judgment-call finding still blocks
auto-merge — carry it to the PR.

**Hard bounds regardless of budget:** the workflow's concurrency cap (~14 agents),
1000 agents per workflow lifetime, and worktree-per-verifier setup cost. The merge
gate is unchanged except that "the verifier passed" now means the *panel's* majority
verdict, and "review clean" means the dimensioned pass left no unresolved /
judgment-call finding.
