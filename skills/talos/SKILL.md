---
name: talos
description: Work a plan.md to completion, one item at a time, unattended — implement, red-team, judge, commit, simplify, repeat. Use when the user says "run talos", "run the loop", "work the plan", "start the harness", or asks to grind through plan.md overnight.
---

# talos — the sequential plan harness

> Talos walked the same circuit of Crete three times a day, unattended, until
> he was stopped. That is the whole design.

You are the **driver**. You do not write the code; you run the procedure. Four
rules govern everything below:

1. **Sequential.** One item at a time. The tree is clean between items.
2. **The maker is never the checker.** The red cell sees the item and the diff,
   never the implementer's account of its own work.
3. **Only the driver writes state.** Every `plan.md` / `decisions.md` edit and
   every `git commit` is yours. Subagents return text; you act on it.
4. **Fail closed.** A reply you cannot parse a `VERDICT:` line out of is a
   failure, and it consumes an attempt.

**Arguments.** `/talos [plan-path] [implementer-pin] [checker-pin]`, any order.
A pin is `<model>-<effort>` (e.g. `opus-medium`, `fable-high`); every other
argument is the plan path.

- **Plan path.** Default `plan.md` (`/talos-mini` passes `mini-plan.md`).
  `decisions.md` is always `decisions.md`. Everything below says "the plan"
  for the resolved path.
- **Implementer pin.** The first pin. Default `opus-medium`.
- **Checker pin.** The second pin, used for the red cell and the judge on every
  pass. Default `opus-medium`.

Spawn with `model: <model>`, `effort: <effort>` from the pin. Echo the resolved
pins once at start of session.

**Spawning subagents.** Use the Agent tool with `subagent_type:
"general-purpose"`, `run_in_background: false` (you need the result before you
can continue), and the pin for the role. The prompt bodies are in
`prompts/implementer.md`, `prompts/red-cell.md`, `prompts/judge.md` — read the
one you need, fill its placeholders, and pass the filled text as the prompt.
Every subagent gets a *fresh* context: never carry one role's transcript into
another.

## Start of session

1. `git status --porcelain` must be empty. If it is not, stop and ask — do not
   start on a dirty tree.
2. Read the plan. Take the goal from line 1 and the **test command** from line
   2; that command is the gate for every item this session.
3. Read the last ~20 lines of `decisions.md` (if it exists) and
   `git log --oneline -5`. That is your entire history. Do not go looking for
   more.
4. Confirm the test command passes right now. If it already fails, stop and
   report — you cannot verify anything against a red baseline.

## Iteration

At the top of every iteration, check the stop conditions:

- No `[ ]` items left → **final report**, stop.
- **3 consecutive blocked items** → systemic problem. Stop with a summary; do
  not keep grinding.
- **8–10 items done this session** → finish the current item, verify the tree
  is clean, and stop with: *"clean checkpoint — /clear and re-run /talos to
  continue."* All state is in the plan, `decisions.md`, and git, so this is a
  free resume.
- A user interrupt is always safe between items.

Otherwise:

### 0. Hygiene

`git status --porcelain` must be empty. If it is dirty, `git checkout . && git
clean -fd` and note in one line what you discarded. Never begin an item on a
dirty tree.

### 1. Pick the item

The **first** `[ ]` line in the plan, in file order. Note its number `N` and
its full text including any indented detail block.

### 2. Implement — implementer pin

Fill `prompts/implementer.md` and spawn. It gets: the item, the whole plan for
context, the test command, and — on a retry — the previous red-cell findings
and the judge's fix instructions.

Read back `SUMMARY:`, `TESTS:`, and optional `PLAN-IMPACT:`. Keep the
`PLAN-IMPACT` text for step 4; **discard the summary** — it must not reach the
red cell.

If the implementer made no changes at all (`git diff` and `git status` both
empty):

- **with** a `PLAN-IMPACT` — it is telling you the item is unactionable. Skip
  step 3 (there is nothing to attack) and go straight to step 4 with an empty
  diff and the proposal. This is the normal route to REPLAN and costs no
  attempt.
- **without** one — go to step 5's FIX-REQUIRED path with "no changes were
  made" as the instruction. This consumes an attempt.

### 3. Red cell — checker pin

Run `git diff` (plus `git status --porcelain` to catch new files; `git add -N .`
first so untracked files appear in the diff). Fill `prompts/red-cell.md` with
the item text and the diff **inline**. Spawn a fresh agent.

Parse `VERDICT: PASS|FAIL` and `FINDINGS:`. Unparseable → treat as FAIL with
"red cell returned no parseable verdict" as the sole finding.

### 4. Judge — checker pin, always

Runs whether the red cell passed or failed. Fill `prompts/judge.md` with: the
item, the same diff, the red cell's verdict and findings, and — if the
implementer returned one — its `PLAN-IMPACT` proposal, labelled as a proposal
from the implementer.

Parse `VERDICT: APPROVE|FIX-REQUIRED|REPLAN`, `RULING:`, and the conditional
`FIX-INSTRUCTIONS:` / `PLAN-EDITS:`. Unparseable → treat as FIX-REQUIRED with
"judge returned no parseable verdict" as the instruction.

### 5. Act on the verdict

**APPROVE**

1. Mark item `N` as `[x]` in the plan.
2. Apply `PLAN-EDITS` if the judge gave any.
3. If the red cell said FAIL, append a `decisions.md` entry recording the
   override and the judge's rebuttal. Also append one for any `PLAN-EDITS`.
4. Commit code + plan + `decisions.md` as **one** commit:
   `feat(<scope>): <item, imperative> (plan item N)`
   Use `fix(...)` / `chore(...)` where that fits the item better. One commit
   never contains two items.
5. Go to step 6.

**FIX-REQUIRED**

Retry from step 2, passing the fix instructions and the red-cell findings as
feedback. Leave the failed attempt's changes in the tree — the implementer
repairs them in place. **Cap: 3 attempts total per item.** On the 3rd failure,
go to the blocked flow.

**REPLAN**

1. `git checkout . && git clean -fd` — the attempt is discarded entirely.
2. **Then** apply the judge's `PLAN-EDITS` to the plan.
3. Append a `decisions.md` entry with the reasoning.
4. Commit: `chore(plan): replan item N — <reason>`.
5. Return to step 1 **without** counting an attempt against the new items.
   A single item may be replanned at most twice; a third replan is a block.

**Order matters, and getting it wrong is silent.** `git checkout .` reverts
tracked files, and the plan is tracked — so discarding *after* you edit the
plan throws your edit away and you commit an unchanged plan. The item then
comes back around next iteration and the loop spins. Discard first, edit
second, always. Before committing, re-read the plan and confirm your edit is
actually in it.

If after applying `PLAN-EDITS` item `N` is still `[ ]` with unchanged text, the
judge has not actually changed anything — treat it as a block rather than
looping.

### 6. Simplify

Run the `/simplify` skill over the changes in the commit you just made. If it
changed nothing, skip to step 9.

### 7. Light re-verify — only if simplify changed something

1. Run the test command. Failure → discard (step 7.3).
2. One red-cell pass, **checker pin**, scoped to the simplification diff only
   (`git diff` against the item commit), with the item text as the contract.
3. Tests pass **and** red cell PASS → step 8. Red cell FAIL → spawn the judge
   (checker pin) on the simplification diff. APPROVE → step 8. Anything else
   → `git checkout . && git clean -fd`.

**Never retry a simplification.** A discarded simplification is not a failure
of the item and gets no `decisions.md` entry — the item is already committed
and correct.

### 8. Commit the simplification separately

`refactor(<scope>): simplify item N changes`

### 9. Continue

Emit one ledger line — `item N: approved in K attempts` / `item N: blocked` /
`item N: replanned` — and start the next iteration. Do not summarise the item's
content; the commit and `decisions.md` hold that.

## Blocked flow

On the 3rd failed attempt at an item:

1. `git checkout . && git clean -fd`. **Reset, never stash** — a stash queue
   accumulating across an overnight run is a landmine. The attempt's knowledge
   is preserved in `decisions.md` instead, which is the point of writing it.
2. **Then** mark the item `[!]` with a one-line reason and a
   `(see decisions.md #D<n>)` pointer — same ordering trap as REPLAN: the reset
   must happen before you touch the plan.
3. Append a `decisions.md` entry: the final red-cell findings, the judge's last
   fix instructions, and what each of the 3 attempts tried. This is what the
   user reads in the morning to unblock it.
4. Commit: `chore(plan): block item N — <reason>`.
5. Continue to the next item — unless this is the 3rd consecutive block, in
   which case stop.

## Context hygiene

Each iteration reads **only**: the plan, the last ~20 lines of `decisions.md`,
and `git log --oneline -5`. Never re-read a past subagent's transcript, never
carry an earlier item's diff forward, never re-read files you touched three
items ago. If you find yourself wanting the history, it is in git.

## Final report

When the plan has no `[ ]` items left, or you stop early, report:

- items completed, blocked, and remaining, by number;
- the `decisions.md` entries written this session, one line each;
- `git log --oneline` for the session's commits;
- confirmation that `git status --porcelain` is empty and the test command
  passes.

Then stop. Do not start unrequested follow-up work.
