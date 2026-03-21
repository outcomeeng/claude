---
name: testing
description: >-
  ALWAYS invoke this skill when creating tests from assertions, running tests, or checking stale status.
  NEVER create or run spec tree tests without this skill.
allowed-tools: Read, Glob, Grep, Write, Edit, Bash
---

<!-- PLACEHOLDER: Full implementation in Phase 3 -->

<objective>

Create tests from assertion links in specs, invoke language-specific testing skills, run tests, and detect stale nodes lacking evidence.

Operates in two modes:

- **Direct invocation:** User asks to create or run tests.
- **Postflight:** Action skills trigger after making changes to synchronize spec-test consistency.

</objective>
