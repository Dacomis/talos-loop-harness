---
name: talos-mini
description: Turn the tasks agreed in this conversation into a mini-plan.md and run the sequential loop over them. Use when the user says "go do those", "run talos on that", "talos-mini", or hands over a written brief of tasks instead of a formal plan.md.
---

# talos-mini — ad-hoc task lists

Same engine as `/talos`, different item source. For the case where a handful of
tasks were just agreed in conversation and writing a formal `plan.md` in a
separate session would be ceremony.

## 1. Distil

Write `mini-plan.md` at the repo root, in the exact grammar of
`../talos/FORMAT.md`:

```markdown
# Plan: <what this batch of work is for>
Test command: `<the project's test command>`

- [ ] 1. <imperative task>
  <indented detail block: paths, constraints, acceptance criteria>
```

Determine the test command from the repo (`package.json` scripts,
`Makefile`, `composer.json`, …), not from memory. If there is genuinely no test
suite, use a command that at least type-checks or lints, and say so.

**Write every item self-contained.** This is the one thing that makes or breaks
a mini-plan. The loop's subagents — and any resume after `/clear` — see the
plan file and nothing else. This conversation does not exist to them. So every
file path, branch name, API name, constraint, rejected alternative, and
acceptance criterion that was established in the discussion must appear in the
item's line or its detail block. An item that reads "as we discussed", "the
same fix as above", or "in that file" is broken.

## 2. Confirm

Show the drafted `mini-plan.md` and ask for a go-ahead — unless the user
already said "just do it". On OK, commit it:
`chore(plan): mini-plan for <batch>`.

## 3. Run

Invoke `/talos` with the plan path `mini-plan.md`, plus any pins passed to
`/talos-mini` (e.g. `/talos-mini fable-high opus-medium`). The per-item procedure is
identical and unmodified; decisions still append to `decisions.md`.

## Resuming and reuse

`mini-plan.md` stays committed as the record of the run. On the next
`/talos-mini`, read it first:

- If it still has `[ ]` items, **resume it** — do not overwrite. Ask the user
  whether to finish it or to append the new tasks to it.
- If every item is `[x]` or `[!]`, overwrite it with the new batch.

## Working from a written brief

If the user pastes a brief written elsewhere (typically by another session that
ended with "here is the work for a fresh conversation"), treat the brief as the
conversation: distil it into `mini-plan.md` the same way, applying the same
self-containment rule to whatever the brief left implicit.
