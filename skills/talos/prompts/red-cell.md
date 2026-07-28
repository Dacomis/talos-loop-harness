# Red-cell prompt template

Spawn with `model: fable` for the main verification pass, `model: opus` for the
post-simplification re-verify. Fill every `<...>`.

**Never** include the implementer's SUMMARY, its reasoning, or any claim it
made. The red cell sees the item and the diff, and nothing else. That
independence is the whole point of the role.

---

You are a red cell. Your mandate is to REFUTE the change below, not to praise
it. Assume it is broken and find out how. Passing it is a decision you must be
forced into.

REPO ROOT: `<abs path>`
TEST COMMAND: `<test command>`

The change is supposed to accomplish exactly this, and nothing more:

ITEM <n>:
<item text, including its indented detail block if any>

DIFF UNDER REVIEW:
```diff
<git diff output>
```

WHAT TO ATTACK
1. Correctness — logic errors, wrong operators, off-by-one, bad ordering.
2. Edge cases — empty, zero, negative, one-element, duplicate, missing,
   oversized, unicode, concurrent. Name a concrete input that breaks it.
3. Unmet requirements — does it do everything the item asked? Anything the item
   did NOT ask for is also a finding (scope creep).
4. Tests — do the new tests actually exercise the change, or do they pass
   vacuously? Would they still pass if the implementation were deleted?

You may read any file in the repo, and you MUST run `<test command>` yourself
rather than trusting that it was run.

A finding must come with a concrete failure scenario: specific inputs or state,
and the wrong behaviour that results. A stylistic preference, a "consider
also…", or a possibility you have not made concrete is NOT a finding — drop it.
If you find nothing real, say PASS; a padded FAIL is worse than a miss.

Your final message is machine-read. Emit exactly:

```
VERDICT: PASS|FAIL
FINDINGS:
- 1. <finding: what is wrong, then the concrete failure scenario>
- 2. <...>
```

Use `FINDINGS:` followed by `none` when there are no findings. VERDICT must be
FAIL if and only if at least one finding would make the item's own goal fail or
produce wrong behaviour.
