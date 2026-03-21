---
name: coding
description: >-
  ALWAYS invoke this skill before implementing any spec-tree work item.
  NEVER write implementation code without following this TDD flow.
---

<objective>
Orchestrate the spec-tree TDD flow for a work item. Eight phases, strictly sequential. Three review gates that loop until APPROVED — no exceptions, no soft passes.

</objective>

<quick_start>

1. Detect project language
2. Load methodology (Phase 1 — once per session)
3. Load work item context (Phase 2 — every node)
4. Architect → review until APPROVED (Phases 3–4)
5. Test → review until APPROVED (Phases 5–6)
6. Implement → review until APPROVED (Phases 7–8)

</quick_start>

<language_detection>
Determine the project language before proceeding. This selects which skills to invoke in Phases 3–8.

| Indicator                       | Language   | Plugin prefix |
| ------------------------------- | ---------- | ------------- |
| `pyproject.toml`, `*.py`        | Python     | `python`      |
| `tsconfig.json`, `package.json` | TypeScript | `typescript`  |

If both are present or neither is found, ask the user.

</language_detection>

<skill_map>
Reference table for all phase invocations. Replace `{P}` with the detected plugin prefix.

| Phase | Purpose                | Skill to invoke                         |
| ----- | ---------------------- | --------------------------------------- |
| 1     | Load methodology       | `spec-tree:understanding`               |
| 2     | Load work item context | `spec-tree:contextualizing`             |
| 3     | Architect              | `{P}:architecting-{language}`           |
| 4     | Architecture review    | `{P}:reviewing-{language}-architecture` |
| 5     | Write tests            | `{P}:testing-{language}`                |
| 6     | Test review            | `{P}:reviewing-{language}-tests`        |
| 7     | Implement              | `{P}:coding-{language}`                 |
| 8     | Code review            | `{P}:reviewing-{language}`              |

**Concrete examples:**

| Phase | Python                                 | TypeScript                                     |
| ----- | -------------------------------------- | ---------------------------------------------- |
| 3     | `python:architecting-python`           | `typescript:architecting-typescript`           |
| 4     | `python:reviewing-python-architecture` | `typescript:reviewing-typescript-architecture` |
| 5     | `python:testing-python`                | `typescript:testing-typescript`                |
| 6     | `python:reviewing-python-tests`        | `typescript:reviewing-typescript-tests`        |
| 7     | `python:coding-python`                 | `typescript:coding-typescript`                 |
| 8     | `python:reviewing-python`              | `typescript:reviewing-typescript`              |

</skill_map>

<phases>

<phase number="1" name="Load methodology" frequency="once per session">

Invoke `spec-tree:understanding`.

This loads the spec-tree methodology — node types, assertion formats, durable map rules. Skip if already invoked in this session.

**Do not proceed until complete.**

</phase>

<phase number="2" name="Load work item context" frequency="every node">

Invoke `spec-tree:contextualizing`.

Load the full context hierarchy for the specific node — parent chain, sibling nodes, applicable decisions, assertions.

**Repeat for every new node.** Do not reuse context from a previous node.

**Do not proceed until complete.**

</phase>

<phase number="3" name="Architect">

Invoke the architecting skill for the detected language.

Produce the ADR(s) for the work item. The architecture must be complete before review.

</phase>

<phase number="4" name="Architecture review" gate="true">

Invoke the architecture review skill for the detected language.

**REJECT → fix → re-invoke this phase.** Loop until APPROVED.

</phase>

<phase number="5" name="Write tests">

Invoke the testing skill for the detected language.

Write tests for all assertions in the spec. Tests come before implementation — no exceptions.

</phase>

<phase number="6" name="Test review" gate="true">

Invoke the test review skill for the detected language.

**REJECT → fix → re-invoke this phase.** Loop until APPROVED.

</phase>

<phase number="7" name="Implement">

Invoke the coding skill for the detected language.

Write implementation code. All tests from Phase 5 must pass.

</phase>

<phase number="8" name="Code review" gate="true">

Invoke the code review skill for the detected language.

**REJECT → fix → re-invoke this phase.** Loop until APPROVED.

</phase>

</phases>

<review_gates>

Phases 4, 6, and 8 are blocking gates. The flow does not advance past a review until the verdict is APPROVED.

- APPROVED or REJECT. Nothing else exists.
- "Approved with observations" is not a verdict — observations that matter are rejections.
- Each rejection cycle: fix the findings, then re-invoke the same review skill.
- Do not skip review phases. Do not proceed on REJECT.

</review_gates>

<rationale>
When something breaks or behaves unexpectedly, your instinct will be to write ad hoc code — a quick script, a throwaway snippet, a print-and-pray debugging session. That instinct is the symptom, not the fix. The problem you hit exists because your tests were insufficient. The ad hoc code patches over one instance; a proper test catches every future instance too.

1. **Do not** write ad hoc code to "see what's happening."
2. **Do** write a test that reproduces the problem. The fact that you hit this issue proves your test coverage has a gap.
3. **Then** fix the implementation until the test passes.

This is not slower. The ad hoc script you were about to write takes the same effort as a test, but the script gets deleted and the test stays.

</rationale>

<success_criteria>

- Phase 1 complete: spec-tree methodology loaded
- Phase 2 complete: work item context hierarchy loaded
- Phase 4 verdict: APPROVED (architecture review passed)
- Phase 6 verdict: APPROVED (test review passed)
- Phase 8 verdict: APPROVED (code review passed)
- All tests pass
- No phase skipped, no review gate bypassed

</success_criteria>
