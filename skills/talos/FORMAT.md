# State file formats

Two files, both at the consumer repo root. Nothing else is loop state.

## `plan.md`

```markdown
# Plan: <goal one-liner>
Test command: `pnpm test`

- [ ] 1. Short imperative item description
- [x] 2. Done item
- [!] 3. Blocked item — one-line reason (see decisions.md #D4)
```

- Line 1 is the goal. Line 2 is `Test command:` followed by the exact shell
  command that gates every item. Both are required.
- Markers: `[ ]` todo, `[x]` done, `[!]` blocked. Nothing else.
- Items are numbered for stable reference. Numbers are never reused or
  renumbered; a new item takes the next unused number and is *inserted at the
  position it belongs in the sequence*, so numbers need not be in order.
- One line per item, except that an item may be followed by an indented detail
  block (2 spaces) giving file paths, constraints, or acceptance criteria. Use
  it whenever the one-liner would not be self-contained.

An item must be intelligible to an agent that sees nothing but this file.

## `decisions.md`

Append-only. Newest entries at the bottom.

```markdown
## D<seq> — <YYYY-MM-DD> — item <n>: <title>
<2-6 lines: what was decided or ruled and why, plus any plan.md change made>
```

Write an entry only for things the user should read in the morning:

- a judge ruling that overrides the red cell (or vice versa),
- a plan edit, with the reasoning behind it,
- a blocked item, with the final findings and fix instructions.

Do not log routine approvals, retries, or simplifications — git already has
those.
