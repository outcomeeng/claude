# Test Review Protocol

Determine if tests provide genuine evidence that assertions are fulfilled through adversarial review. Reject tests that can pass while assertions remain unfulfilled.

**THE ADVERSARIAL QUESTION:** How could these tests pass while the assertion remains unfulfilled? If you can answer that question, the tests are **REJECTED**.

## Verdict

There is no middle ground. No "mostly good." No "acceptable with caveats."

- **APPROVED**: Tests provide genuine evidence for all assertions at appropriate levels
- **REJECT**: Any deficiency, missing link, silent skip, or evidentiary gap

A missing comma is REJECT. A philosophical disagreement about test structure is REJECT. If it's not APPROVED, it's REJECT.

## Four-phase review protocol

Execute phases IN ORDER. Stop at first REJECT.

### Phase 1: Spec structure validation

For each assertion in the spec, verify:

**1.1 Assertion format**

Assertions MUST use one of five typed formats. No code in specs.

| Type            | Quantifier                     | Test strategy   | Format pattern                                     |
| --------------- | ------------------------------ | --------------- | -------------------------------------------------- |
| **Scenario**    | There exists (this case works) | Example-based   | `Given ... when ... then ... ([test](...))`        |
| **Mapping**     | For all over finite set        | Parameterized   | `{input} maps to {output} ([test](...))`           |
| **Conformance** | External oracle                | Tool validation | `{output} conforms to {standard} ([test](...))`    |
| **Property**    | For all over type space        | Property-based  | `{invariant} holds for all {domain} ([test](...))` |
| **Compliance**  | ALWAYS/NEVER rules             | Review or test  | `ALWAYS/NEVER: {rule} ([review]/[test](...))`      |

**If spec contains code examples**: REJECT. Specs are durable; code drifts.

**Assertion type must match test strategy:**

| Assertion Type | Required Test Pattern    | REJECT if                                    |
| -------------- | ------------------------ | -------------------------------------------- |
| Scenario       | Example-based tests      | Missing concrete inputs/outputs              |
| Mapping        | Parameterized tests      | Only example-based (not all cases covered)   |
| Property       | Property-based framework | Only example-based (must use property-based) |
| Conformance    | Tool validation          | Manual checks instead of tool                |
| Compliance     | `[review]` or `[test]`   | No tag indicating verification method        |

**1.2 Test file linkage**

Inline test links are contractual. Every `([test](...))` link in an assertion must resolve to an actual file. Stale links = REJECT.

Check:

1. Link syntax is valid Markdown: `[test](path)` or `[display](path)`
2. Linked file EXISTS at specified path
3. Level matches filename convention (language-specific)

**If link is broken or file missing**: REJECT.

**1.3 Level appropriateness**

| Evidence Type              | Minimum Level | Example                                  |
| -------------------------- | ------------- | ---------------------------------------- |
| Pure computation/algorithm | 1             | Protocol timing, math correctness        |
| Component interaction      | 2             | TX→RX loopback, multi-entity simulation  |
| Project-specific binary    | 2             | Verilator lint, external tool invocation |
| Real credentials/services  | 3             | Cloud APIs, payment providers            |

**If assertion is tested at wrong level**: REJECT.

**GATE 1**: All assertions use typed format? Assertion types match test strategy? All test links resolve? All levels appropriate?

### Phase 2: Evidentiary integrity

For each test file, verify it provides genuine evidence.

**2.1 The adversarial test**

Ask: **How could this test pass while the assertion remains unfulfilled?**

| Scenario                                                   | Verdict |
| ---------------------------------------------------------- | ------- |
| Test asserts something other than what assertion specifies | REJECT  |
| Test uses hardcoded values that happen to match            | REJECT  |
| Test doesn't actually exercise the code path               | REJECT  |
| Test mocks the thing it's supposed to verify               | REJECT  |
| Test can pass with broken implementation                   | REJECT  |

**2.2 Dependency availability**

**CRITICAL: Missing dependencies MUST FAIL, not skip.**

| Pattern                                      | Verdict                                          |
| -------------------------------------------- | ------------------------------------------------ |
| Skip on required project dependency          | **REJECT** - Required dependency must fail       |
| Skip on test infrastructure (property lib)   | **REJECT** - Test infrastructure must be present |
| Skip on platform (`sys.platform`, `os.type`) | REVIEW - May be legitimate                       |

**The Silent Skip Problem:** Tests that silently skip on required dependencies allow CI to go green with zero verification. This is evidentiary fraud.

**2.3 Harness verification**

If assertion specifies a harness: harness must exist, have its own tests, and harness failures must cause test failures (not skips).

**GATE 2**: Each test file reviewed adversarially? Skip patterns evaluated? Harnesses verified?

### Phase 3: Lower-level assumptions

Specs are DURABLE. They DEMAND assertions. A spec must NEVER say "stories are pending" or "tests will be added later."

**Atemporal voice** (Durable Map Rule): Specs state product truth. They NEVER narrate code history, current state, or migration plans. Any temporal language is REJECT — no section gets a pass.

Temporal patterns to reject:

- "The current `module.py` has..." — narrates code state
- "We need to replace..." / "We need to migrate..." — narrates a plan
- "Currently X uses..." — snapshot that expires
- "The existing implementation..." — references code, not architecture
- "Previously..." / "Before this..." — there is no before

**Integration test assumptions:** Integration tests verify component contracts and interoperation. They should not duplicate lower-level evidence.

**GATE 3**: No temporal language? Lower-level specs checked? Integration tests not duplicating lower-level work?

### Phase 4: ADR/PDR compliance

Check test code against decision records.

1. Identify applicable ADRs/PDRs (spec references + ancestry)
2. Extract constraints from each
3. Search test files for violations

**If tests violate ADR/PDR constraints**: REJECT.

**GATE 4**: All ADRs/PDRs identified? Each constraint checked? No violations?

## Output format

### APPROVED

```markdown
## Test Review: {container_path}

### Verdict: APPROVED

All assertions have genuine evidentiary coverage at appropriate levels.

### Assertions Verified

| # | Assertion | Type   | Level | Test File | Evidence Quality |
| - | --------- | ------ | ----- | --------- | ---------------- |
| 1 | {name}    | {type} | {N}   | {file}    | Genuine          |

### ADR/PDR Compliance

| Decision Record | Status    |
| --------------- | --------- |
| {name}          | Compliant |
```

### REJECT

```markdown
## Test Review: {container_path}

### Verdict: REJECT

{One-sentence summary of primary rejection reason}

### Rejection Reasons

| # | Category | Location    | Issue   | Required Fix |
| - | -------- | ----------- | ------- | ------------ |
| 1 | {cat}    | {file:line} | {issue} | {fix}        |

### How Tests Could Pass While Assertion Fails

{Explain the evidentiary gap}
```

## Rejection triggers

| Category            | Trigger                                                          | Verdict |
| ------------------- | ---------------------------------------------------------------- | ------- |
| **Spec Structure**  | Code examples in spec                                            | REJECT  |
| **Spec Structure**  | Assertion type doesn't match test strategy                       | REJECT  |
| **Spec Structure**  | Missing or broken test file links                                | REJECT  |
| **Spec Structure**  | Language about "pending" specs                                   | REJECT  |
| **Spec Structure**  | Temporal language ("currently", "the existing", file references) | REJECT  |
| **Level**           | Assertion tested at wrong level                                  | REJECT  |
| **Dependencies**    | Skip on required dependency                                      | REJECT  |
| **Dependencies**    | Harness referenced but missing                                   | REJECT  |
| **Decision Record** | Test violates ADR/PDR constraint                                 | REJECT  |
| **Evidentiary**     | Test can pass with broken impl                                   | REJECT  |

## Cardinal rule

**If you can explain how the tests could pass while the assertion remains unfulfilled, the tests are REJECTED.**

Your job is to protect the test suite from phantom evidence. A rejected review that catches an evidentiary gap is worth infinitely more than an approval that lets one through.
