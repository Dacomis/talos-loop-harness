# talos-loop-harness

Talos, the bronze automaton, walked the same circuit of Crete three times a day
without supervision. This runs a plan the same way.

A sequential loop for working a plan to completion overnight. One item at a
time, implemented by a subagent, attacked by an independent red cell, ruled on
by an independent judge, then committed by the main session.

## Design

- **Sequential.** One item at a time, clean tree between items. No worktrees,
  no lanes, no merge gate, no requeue.
- **Adversarially verified.** The implementer never grades itself. The red cell
  sees the item text and the diff, never the implementer's account of it. The
  judge runs on every item, including the ones the red cell passed.
- **The main session drives.** All plan edits and all commits are made by the
  driving session, never by a subagent. Subagents return text; the driver acts.
- **State is on disk, not in context.** `plan.md` + `decisions.md` + git
  history are the entire state, so `/clear` and re-invoke resumes for free.

## Files

| Path | What it is |
|---|---|
| `skills/talos/SKILL.md` | The driver procedure, and the bulk of the harness. |
| `skills/talos/FORMAT.md` | `plan.md` / `decisions.md` format spec. |
| `skills/talos/prompts/implementer.md` | Implementer subagent prompt. |
| `skills/talos/prompts/red-cell.md` | Red-cell subagent prompt. |
| `skills/talos/prompts/judge.md` | Judge subagent prompt. |
| `skills/talos-mini/SKILL.md` | Ad-hoc variant: distil a conversation's task list into `mini-plan.md`, then run the same loop. |

## Install into a consumer repo

```bash
mkdir -p <repo>/.claude/skills
cp -r skills/talos      <repo>/.claude/skills/talos
cp -r skills/talos-mini <repo>/.claude/skills/talos-mini
```

Then write a `plan.md` in that repo, in the format given in
`skills/talos/FORMAT.md`. Author it in an ordinary session with a strong model:
a badly written plan is the one failure the loop cannot recover from. Commit
it, then run `/talos`.

`decisions.md` is created by the loop on first need. For task lists agreed in
conversation, run `/talos-mini` and skip the plan-authoring session.

## Reading the output in the morning

- `plan.md`: `[x]` done, `[ ]` not reached, `[!]` blocked.
- `decisions.md`: every judge override, plan edit, and block, with reasoning.
- `git log --oneline`: one `feat:` or `fix:` commit per item, each optionally
  followed by its `refactor:` simplification commit.
