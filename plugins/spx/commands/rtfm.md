---
description: Load specs and testing methodology before writing any code
allowed-tools: Skill
---

<objective>
Gate all implementation work behind two mandatory skill invocations — in order, no exceptions. First load the full spec context hierarchy, then load testing methodology and write tests before touching any implementation code.
</objective>

<context>
**Current Branch:**
!`git branch --show-current`

**Git Status:**
!`git status --short`

**Recent Commits:**
!`git log --oneline -5`
</context>

<process>

## Step 1: Understand the specs

Invoke the understanding skill NOW:

```json
Skill tool → { "skill": "spx:understanding-spx" }
```

This loads the full context hierarchy — PRDs, ADRs, capability and feature specs — for whatever you are about to implement. Without this, you will miss requirements, violate decisions, and produce work that has to be redone.

**Do not proceed to Step 2 until you have completed this step.**

## Step 2: Load testing methodology and write tests first

Invoke the testing skill NOW:

```json
Skill tool → { "skill": "test:testing" }
```

Write proper tests before writing any implementation code.

**Do not write any implementation code until you have written the tests.**

</process>

<rationale>
When something breaks or behaves unexpectedly, your instinct will be to write ad hoc code — a quick script, a throwaway snippet, a print-and-pray debugging session. That instinct is the symptom, not the fix. The problem you hit exists because your tests were insufficient. The ad hoc code patches over one instance; a proper test catches every future instance too.

1. **Do not** write ad hoc code to "see what's happening."
2. **Do** write a test that reproduces the problem. You will need that test again — the fact that you hit this issue proves your test coverage has a gap.
3. **Then** fix the implementation until the test passes.

This is not slower. The ad hoc script you were about to write takes the same effort as a test, but the script gets deleted and the test stays.

</rationale>

<success_criteria>

- Understanding skill (`spx:understanding-spx`) invoked and full context hierarchy loaded
- Testing skill (`test:testing`) invoked and methodology loaded
- Tests written before any implementation code
- If you have concerns about specs or testing approach, raise them NOW — before writing code, not after

</success_criteria>
