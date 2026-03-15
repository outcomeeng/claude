<overview>
Every assertion in an outcome spec must be one of four structured types. The type determines how the assertion is tested.

| Type            | Quantifier                      | Test strategy   | Use when                                      |
| --------------- | ------------------------------- | --------------- | --------------------------------------------- |
| **Scenario**    | There exists (this case works)  | Example-based   | Specific user journey or interaction          |
| **Mapping**     | For all over a finite set       | Parameterized   | Input-output correspondence over known values |
| **Conformance** | External oracle                 | Tool validation | Must match an external standard or schema     |
| **Property**    | For all over a type/value space | Property-based  | Invariant that must hold for all valid inputs |

</overview>

<scenario>

**Quantifier:** There exists — "this specific case works."

A scenario describes a concrete interaction in natural language.

```markdown
- Given a tree with all valid children, when status is computed, then the parent reports valid ([test](tests/status.unit.test.ts))
```

**Test strategy:** Example-based tests. Each scenario maps to one or more test cases with concrete inputs and expected outputs.

**When to use:** User journeys, specific interactions, error cases, edge cases that need explicit coverage.

</scenario>

<mapping>

**Quantifier:** For all over a finite, enumerable set.

A mapping defines input-output correspondence across a known set of values. Often expressed as a table.

```markdown
- Node with no lock file maps to "needs work" ([test](tests/status.unit.test.ts))
- Node with matching blobs maps to "valid" ([test](tests/status.unit.test.ts))
- Node with mismatched blobs maps to "stale" ([test](tests/status.unit.test.ts))
```

**Test strategy:** Parameterized tests. Each row in the mapping becomes a test case.

**When to use:** State machines, lookup tables, enum-to-behavior mappings, finite configuration spaces.

</mapping>

<conformance>

**Quantifier:** External oracle — "must match what this reference says."

A conformance assertion states that output must match an external standard, schema, or reference.

```markdown
- Lock file conforms to spx-lock/v1 schema ([test](tests/schema.unit.test.ts))
- Output conforms to POSIX exit code conventions ([test](tests/exit-codes.unit.test.ts))
```

**Test strategy:** Tool-based validation. Use schema validators, linters, or comparison against reference output.

**When to use:** Schema compliance, format standards, API contracts, protocol conformance.

</conformance>

<property>

**Quantifier:** For all over a type or value space — "this invariant always holds."

A property assertion states something that must be true for all valid inputs, not just specific examples.

```markdown
- Lock file is deterministic: same spec and test content always produces the same lock ([test](tests/lock.unit.test.ts))
- Ordering is transitive: if A constrains B and B constrains C, then A constrains C ([test](tests/ordering.unit.test.ts))
```

**Test strategy:** Property-based testing (e.g., Hypothesis for Python, fast-check for TypeScript). Generate random valid inputs and verify the property holds.

**When to use:** Algebraic invariants, idempotency, commutativity, determinism guarantees, "for all valid X, Y holds."

</property>

<choosing_type>

1. Can you enumerate all cases? → **Mapping**
2. Is there an external reference to match? → **Conformance**
3. Must it hold for all inputs (not just examples)? → **Property**
4. Is it a specific interaction or journey? → **Scenario**

When in doubt, start with **Scenario**. Promote to **Mapping** when you discover the domain is finite. Promote to **Property** when you realize the assertion should hold universally.

</choosing_type>

<mixing_types>

A single outcome spec can contain assertions of different types. Group them by type for readability:

```markdown
### Assertions

- Given a tree with one stale child, parent reports stale ([test](tests/status.unit.test.ts))
- Given a tree with all valid children, parent reports valid ([test](tests/status.unit.test.ts))
- State mapping: no lock = needs-work, matching = valid, mismatched = stale ([test](tests/status.unit.test.ts))
- Status rollup is deterministic: same tree always produces same status ([test](tests/status.unit.test.ts))
```

</mixing_types>
