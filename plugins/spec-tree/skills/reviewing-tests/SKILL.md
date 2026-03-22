---
name: reviewing-tests
description: >-
  ALWAYS invoke this skill when reviewing tests for evidentiary value and spec compliance.
  NEVER review tests without this skill.
---

<objective>
Adversarial review of test evidence against spec-tree assertions. This skill is a superset of the standalone review protocol — it incorporates the full 4-phase adversarial review and adds spec-tree-specific concerns: tree-level coverage, cross-cutting assertion review, orphan detection, and decision record compliance from the full ancestor chain.

</objective>

<quick_start>

**PREREQUISITE**: Read the review protocol reference before reviewing:

- `${SKILL_DIR}/references/review-protocol.md` — 4-phase protocol, verdict rules, rejection triggers

Then follow the spec-tree review workflow below. Language-specific skills (`/reviewing-python-tests`, `/reviewing-typescript-tests`) add language-specific phases on top of this.

</quick_start>

<spec_tree_review_workflow>

## Step 1: Load tree context

Check for `<SPEC_TREE_FOUNDATION>` and `<SPEC_TREE_CONTEXT>` markers. If absent, invoke `/understanding` and `/contextualizing` first.

This loads:

- The target spec node and its assertions
- Ancestor ADRs/PDRs (full chain, not just what you manually find)
- Lower-index sibling specs
- Lower-level child specs (if any)

## Step 2: Execute the four foundational review phases

Execute the 4-phase protocol from `${SKILL_DIR}/references/review-protocol.md` in order. Stop at first REJECT.

1. **Phase 1: Spec structure validation** — Assertion format, test links exist, level appropriateness
2. **Phase 2: Evidentiary integrity** — Adversarial test, dependency handling, harness verification
3. **Phase 3: Lower-level assumptions** — Node hierarchy, atemporal voice, integration boundaries
4. **Phase 4: ADR/PDR compliance** — Decision record constraints from the **full ancestor chain** loaded via tree context (not just ADRs you happen to find)

Use language-specific grep patterns when checking for mocking, skip patterns, etc.

## Step 3: Tree-level coverage analysis

After the foundational phases pass, verify coverage across the tree:

**3.1 Assertion coverage**

Walk the target node (and subtree if reviewing a parent node). For every assertion across all nodes in scope:

| Check                                | REJECT if                                                         |
| ------------------------------------ | ----------------------------------------------------------------- |
| Assertion has `([test](...))` link   | Link missing                                                      |
| Linked test file exists              | File missing                                                      |
| Assertion type matches test strategy | Mismatch (e.g., Property assertion with only example-based tests) |

**3.2 Orphan detection**

List all test files in the `tests/` directory. For each file, verify it is linked from at least one assertion. Report orphaned test files — they may indicate:

- Stale tests from deleted assertions
- Tests that should be linked but aren't
- Tests that belong in a different node

Orphans are not automatic REJECT, but they must be reported.

**3.3 Cross-cutting assertion review**

For assertions at ancestor nodes:

- Verify evidence is provided at the appropriate place (ancestor `tests/` or child `tests/`)
- If a child node provides evidence for a parent assertion, the parent assertion should link to it
- If an ancestor accumulates many cross-cutting assertions without dedicated tests, flag for potential enabler extraction

## Step 4: Invoke language-specific review

After tree-level review passes, invoke the language-specific review skill for additional phases:

- `/reviewing-python-tests` — Property-based testing, Python conventions
- `/reviewing-typescript-tests` — TypeScript-specific patterns

These skills add language-specific rejection triggers (mocking patterns, type annotations, etc.).

## Step 5: Issue verdict

Binary verdict. No middle ground.

**APPROVED** — All foundational phases passed, tree-level coverage complete, language-specific checks passed. Use the APPROVED template from the review protocol.

**REJECT** — Any deficiency at any phase. Use the REJECT template from the review protocol. Include:

- File:line locations for each rejection reason
- How tests could pass while assertion fails (the adversarial explanation)
- Tree-level findings (orphans, coverage gaps, cross-cutting issues)

</spec_tree_review_workflow>

<additional_rejection_triggers>

These triggers apply in addition to the foundational triggers from the review protocol:

| Category           | Trigger                                                        | Verdict |
| ------------------ | -------------------------------------------------------------- | ------- |
| **Tree Coverage**  | Assertion in scope has no test link                            | REJECT  |
| **Tree Coverage**  | Test link present but file doesn't exist                       | REJECT  |
| **Cross-cutting**  | Parent assertion with no evidence path (neither own nor child) | REJECT  |
| **ADR/PDR (tree)** | Ancestor ADR/PDR constraint violated (found via tree context)  | REJECT  |

</additional_rejection_triggers>

<success_criteria>

Review is complete when:

- [ ] Tree context loaded (foundation markers present)
- [ ] All 4 foundational review phases executed (or stopped at first REJECT)
- [ ] Tree-level coverage analyzed for all assertions in scope
- [ ] Orphan test files identified and reported
- [ ] Cross-cutting assertions reviewed for evidence placement
- [ ] ADR/PDR compliance checked from full ancestor chain
- [ ] Language-specific review phases executed
- [ ] Verdict is APPROVED or REJECT (no middle ground)
- [ ] Each rejection has file:line location and adversarial explanation

</success_criteria>
