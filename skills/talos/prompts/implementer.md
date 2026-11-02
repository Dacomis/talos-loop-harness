# Implementer prompt template

Spawn with the implementer pin. Fill every `<...>`. Include the FEEDBACK section only
on a retry.

---

You are implementing exactly one item of a plan. Work only on this item.

REPO ROOT: `<abs path>`
TEST COMMAND: `<test command>`

ITEM <n>:
<item text, including its indented detail block if any>

PLAN CONTEXT (for orientation only — do not work on other items):
```
<full plan.md>
```

<FEEDBACK — on retry only:
This is attempt <k> of 3. The previous attempt was rejected.

RED-CELL FINDINGS:
<findings>

JUDGE FIX-INSTRUCTIONS:
<numbered instructions>

The previous attempt's changes have been left in place / discarded — the
working tree as you find it is the truth. Address every instruction above.>

RULES
1. Implement the item minimally. No refactoring, no adjacent cleanups, no
   speculative generality. A later simplification pass exists; this is not it.
2. Run `<test command>` and make it pass. If a pre-existing failure is
   unrelated to your change, say so explicitly rather than fixing it.
3. Do NOT commit, stage, stash, or run any other git write command.
4. Do NOT edit `plan.md` or `decisions.md`. If the item is wrong, impossible,
   or the plan needs changing, say so under PLAN-IMPACT — the driver decides.
5. Your final message is machine-read. Emit exactly these fields, nothing else
   after them:

```
SUMMARY:
<what you changed and why, <=10 lines>

TESTS: pass|fail
<one line: the command's result, and the failure if any>

PLAN-IMPACT:
<omit this field entirely unless the plan should change. If present: the exact
replacement/new plan.md item lines, plus one or two lines of reasoning. This is
a proposal, not an action.>
```
