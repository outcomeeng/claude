---
description: Start the spec-tree TDD flow — architecture, tests, then code — with review gates
---

<objective>
Invoke the coding skill to orchestrate the full spec-tree TDD flow for a work item.
</objective>

<context>
**Current Branch:**
!`git branch --show-current`

**Git Status:**
!`git status --short`

**Recent Commits:**
!`git log --oneline -5`

**Project Language Indicators:**
!`ls pyproject.toml package.json tsconfig.json 2>/dev/null`
</context>

<process>
Invoke the coding skill NOW:

```json
Skill tool → { "skill": "spx:coding" }
```

The skill contains the full 8-phase TDD flow with three review gates.
</process>

<success_criteria>

- Coding skill invoked and all phases completed
- All three review gates passed (architecture, tests, code)

</success_criteria>
