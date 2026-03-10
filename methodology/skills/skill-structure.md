# Skill Structure

## Plugin: `spec-tree`

All skills in this document belong to the `spec-tree` plugin. Skill names have no suffix — they are invoked as `/understanding`, `/authoring`, etc. (or fully qualified as `spec-tree:understanding`).

## Design principles

- Keep very few skills organized in three layers: foundation, action, contract.
- Foundation skills load once per conversation using marker pattern (no persistent state).
- `understanding` is the single shared library: methodology, structure, templates.
- `contextualizing` handles deterministic context injection from tree structure.
- `testing` enforces spec-test consistency via lock files, both as user entry point and as postflight gate.
- Action skills check for foundation markers before working; invoke foundations if absent.
- Make conversational flow explicit and consistent across action skills.
- Keep migration concerns in a separate optional structure document.

## Spec Tree methodology

The Spec Tree is a git-native product structure. All skills implement and maintain this structure.

### Node types

Two node types replace the former capability/feature/story hierarchy:

| Node type   | Directory suffix | Spec header  | Purpose                                                            |
| ----------- | ---------------- | ------------ | ------------------------------------------------------------------ |
| **Enabler** | `.enabler`       | `## Enables` | Infrastructure that would be removed if all its dependents retired |
| **Outcome** | `.outcome`       | `## Outcome` | Hypothesis about what change a behavior will produce               |

Nodes are nestable at any depth. The tree is not limited to three levels.

### Spec format

Every node directory contains:

- `{slug}.md` -- the spec file (no type suffix, no numeric prefix)
- `tests/` -- co-located test files
- `spx-lock.yaml` -- lock file binding spec to test evidence

Enabler specs start with `## Enables`. Outcome specs start with `## Outcome` followed by `### Assertions` listing test links:

```markdown
## Outcome

We believe that [hypothesis].

### Assertions

- Assertion text ([test](tests/file.unit.test.ts))
```

Every assertion must link to at least one test file. The `spx` CLI parses the Markdown AST to extract and validate these links.

### Product file

The root of every tree is `{product-name}.product.md`, capturing why the product exists and what change in user behavior it aims to achieve. This replaces the former PRD at product level.

### Decision records

PDRs (product constraints) and ADRs (architecture choices) are co-located at any directory level. Their numeric prefix encodes dependency scope within that directory:

```text
spx/
├── product-name.product.md
├── 15-constraint-name.pdr.md
├── 15-technical-choice.adr.md
└── 21-first-enabler.enabler/
```

### Sparse integer ordering

Numeric prefixes encode dependency order within each directory. A lower-index item constrains every sibling with a higher index -- and that sibling's descendants. Items sharing the same index are independent of each other.

Distribution formula for N expected items across range [10, 99]:

```text
i_k = 10 + floor(k * 89 / (N + 1))
```

For N=7: sequence 21, 32, 43, 54, 65, 76, 87.

Fractional indexing (e.g., `20.5-slug`) is the escape hatch when integer gaps are exhausted.

### Lock files and drift detection

`spx-lock.yaml` binds spec content to test evidence via Git blob hashes:

```yaml
schema: spx-lock/v1
blob: a3b7c12
tests:
  - path: tests/file.unit.test.ts
    blob: 9d4e5f2
```

- Edit the spec: blob hash changes, node is visibly stale before tests run.
- Lock files are deterministic: same state produces same file. Two agents produce identical locks.
- Lock is written only when all tests pass.
- `spx-lock.yaml` replaces `status.yaml`. Node state is derived from lock state, not assigned.

Node states derived from lock:

| State          | Condition                                 | Required action          |
| -------------- | ----------------------------------------- | ------------------------ |
| **Needs work** | No lock file exists                       | Write tests, then lock   |
| **Stale**      | Spec or test blob changed since last lock | Re-lock (`spx lock`)     |
| **Valid**      | All blobs match                           | None -- evidence current |

### Deterministic context injection

The tree path from product root to target node defines what context an agent receives. At each directory along the path, all lower-index siblings' specs are injected. Ancestor specs along the path are always included. Test files are excluded.

This replaces heuristic context selection (keyword search, embedding similarity). The agent sees exactly the context the tree provides.

If the deterministic context payload for a node routinely exceeds an agent's reliable working set, the tree signals that the component needs further decomposition.

### Cross-cutting assertions

When a behavior spans multiple nodes, the assertion lives in the lowest common ancestor. If an ancestor accumulates too many cross-cutting assertions, extract a shared enabler at a lower index.

## Intent model (use cases)

### 1. Understand Spec Tree context

1a. Systematically ingest context to prepare for a discussion with the user.
1b. Systematically ingest context to prepare for autonomous work.

### 2. Author Spec Tree artifacts

2a. Author from scratch from user conversation/prompt, including clarifying questions.
2b. Extend existing artifacts with new requirements, outcomes, or decisions.

### 3. Decompose Spec Tree artifacts

3a. Systematically decompose existing higher-level nodes to lower levels.

### 4. Refactor Spec Tree artifacts

4a. Review and structurally refactor (move/re-scope content) through user conversation.
4b. Factor common aspects into shared enablers at lower indices.

### 5. Align Spec Tree artifacts

5a. Clarify/augment/align/deconflict artifacts while preserving product truth.

### 6. Lock file lifecycle based on Spec Tree context

6a. Create tests from assertions in existing specs.
6b. Refactor tests when assertions or decisions change.
6c. Update spec artifacts with test file references and lock when new test evidence reveals gaps.

## Skill map

### Foundation layer

Foundation skills load once per conversation. They emit conversation markers so other skills can detect whether foundation context is present.

| Skill             | Owns                                                                                              | Marker                             |
| ----------------- | ------------------------------------------------------------------------------------------------- | ---------------------------------- |
| `understanding`   | Methodology, durable map worldview, decomposition semantics, ordering rules, all shared templates | `<SPEC_TREE_FOUNDATION>`           |
| `contextualizing` | Deterministic context injection from tree structure, path validation, abort/remediation           | `<SPEC_TREE_CONTEXT target="...">` |

### Action layer

Action skills do the work. Before starting, they check conversation history for foundation markers and invoke missing foundations.

| Skill         | Use case | Scope                                                           |
| ------------- | -------- | --------------------------------------------------------------- |
| `authoring`   | 2        | Create/extend product/ADR/PDR/enabler/outcome from conversation |
| `decomposing` | 3        | Systematically decompose higher-level nodes to lower levels     |
| `refactoring` | 4        | Structural moves, re-scoping, factoring shared enablers         |
| `aligning`    | 5        | Clarify, augment, align, deconflict while preserving truth      |

### Contract layer

The contract skill is both a user-invocable entry point (use case 6) and a postflight gate that action skills trigger after making changes.

| Skill     | Use case | When invoked                                                 |
| --------- | -------- | ------------------------------------------------------------ |
| `testing` | 6        | User invokes directly OR action skills trigger after changes |

## Foundation ownership model

- **`understanding`** is the single shared library for all Spec Tree knowledge:
  - Durable map worldview (specs are permanent product documentation)
  - Decomposition semantics (enabler vs outcome, nesting depth, when to extract shared enablers)
  - Structure and sparse integer ordering rules
  - All shared templates (product, ADR, PDR, enabler, outcome)
  - Template access instructions
- **`contextualizing`** owns deterministic context injection:
  - Walks the tree from product root to target node
  - At each directory along the path, injects all lower-index siblings' specs
  - Validates artifact existence along the path
  - Returns context manifest or abort with remediation
  - Bootstrap mode: returns empty manifest with `bootstrap=true` when authoring into an empty tree (no abort)
- **`testing`** owns lock file lifecycle:
  - Lock file binding between spec content and test evidence
  - Invokes `/testing-[language]` for language-specific test design
  - Runs tests and writes `spx-lock.yaml` only when all pass
  - Updates spec assertion links and flags outcomes lacking evidence
  - Detects stale nodes (spec or test blob changed since last lock)
- **Action skills do not duplicate foundation content.** They reference `understanding` for templates and methodology.

## Marker-based state detection

Foundation skills emit XML markers into the conversation when loaded. Action skills search conversation history for these markers before starting work. This follows the same pattern as `/pickup` emitting `<PICKUP_ID>` for `/handoff` to find.

| Marker                                   | Emitted by        | Checked by                     | Meaning                              |
| ---------------------------------------- | ----------------- | ------------------------------ | ------------------------------------ |
| `<SPEC_TREE_FOUNDATION>`                 | `understanding`   | All action and contract skills | Methodology and templates are loaded |
| `<SPEC_TREE_CONTEXT target="full/path">` | `contextualizing` | All action and contract skills | Target artifacts are loaded          |

**Decision rule:**

- No `<SPEC_TREE_FOUNDATION>` in conversation: invoke `understanding`
- No `<SPEC_TREE_CONTEXT>` matching current target: invoke `contextualizing`
- Target path changed since last `<SPEC_TREE_CONTEXT>`: re-invoke `contextualizing`

## Template ownership

`understanding` owns all templates. Action skills access them via the foundation skill's base directory:

```text
${UNDERSTANDING_DIR}/
├── SKILL.md
├── references/
│   ├── durable-map.md
│   ├── decomposition-semantics.md
│   ├── node-types.md
│   ├── assertion-types.md
│   ├── ordering-rules.md
│   └── what-goes-where.md
└── templates/
    ├── product/
    │   └── product-name.product.md
    ├── decisions/
    │   ├── decision-name.adr.md
    │   └── decision-name.pdr.md
    └── nodes/
        ├── enabler-name.md
        └── outcome-name.md
```

Action skills reference templates with: `Read: ${UNDERSTANDING_DIR}/templates/nodes/outcome-name.md`

## Conversational flow contract

Every action skill follows this interaction contract:

1. **Intake** -- Ask for target path/scope and intended operation.
2. **Foundation gate** -- Check for `<SPEC_TREE_FOUNDATION>` marker; invoke `understanding` if absent.
3. **Target context gate** -- Check for `<SPEC_TREE_CONTEXT>` matching target; invoke `contextualizing` if absent or mismatched. Context is injected deterministically from tree structure. Abort with explicit remediation if required artifacts are missing.
4. **Plan** -- Present concise execution plan and expected outputs.
5. **Execute** -- Perform workflow steps. Keep user in the loop at major decision points.
6. **Contract gate** -- Invoke `testing` to lock spec-test consistency.
7. **Deliver** -- Summarize changes, decisions, and next actions.

## Mode-specific flows

Each flow documents only what is unique to that mode. All action skills share the standard preflight (steps 1-3) and postflight (steps 6-7) from the conversational flow contract above.

### `understanding`

1. Load Spec Tree methodology, structure semantics, and template index.
2. Emit `<SPEC_TREE_FOUNDATION>` marker with loaded module summary.

### `contextualizing`

1. Intake target path/scope and operation type.
2. Walk tree from product root to target node.
3. At each directory along the path, collect lower-index siblings' specs.
4. Include ancestor specs along the path. Exclude test files.
5. Validate collected artifacts exist and are readable.
6. If operation is `author` and no artifacts exist at target level, return empty manifest with `bootstrap=true` instead of aborting.
7. Emit `<SPEC_TREE_CONTEXT target="full/path">` with context manifest: collected specs, lock states, open decisions, readiness status.

### `authoring`

1. Intake node type (enabler or outcome), intended location, and path.
2. Clarify user intent and unresolved product decisions.
3. Draft artifact using templates from `understanding` and Spec Tree rules.
4. Validate atemporal voice, consistency, and testability (assertions link to test files for outcomes).
5. Return draft, open decisions, and recommended next steps (decomposition or locking).

### `decomposing`

1. Intake source node and target decomposition depth.
2. Apply decomposition methodology (enabler vs outcome, scope, sparse integer ordering).
3. Produce child nodes with explicit boundaries and dependencies.
4. Validate decomposition quality (no excessive nesting, correct node types, no misplaced assertions).
5. Return decomposition output with rationale for splits and boundaries.

### `refactoring`

1. Intake structural change request (move, re-scope, extract shared enabler).
2. Analyze impact across hierarchy and decision records.
3. Propose structural change set (moves, consolidations, new enabler nodes).
4. Apply refactoring updates. Re-lock affected nodes.
5. Validate cross-node consistency after structural changes.

### `aligning`

1. Intake alignment request (clarify, augment, deconflict).
2. Analyze contradictions, gaps, or ambiguities across affected nodes.
3. Propose alignment changes with rationale.
4. Apply clarification or deconfliction updates.
5. Validate cross-node consistency and report unresolved conflicts.

### `testing`

When invoked directly (use case 6):

1. Intake target node(s) and language.
2. Invoke `/testing-[language]` for test-level design and implementation.
3. Create or refactor tests as assertion evidence.
4. Run tests and write `spx-lock.yaml` when all pass.
5. Update spec assertion links and flag outcomes lacking evidence.
6. Return evidence summary (what is locked, what is stale, what has no lock).

When invoked as postflight by action skills:

1. Receive change summary from calling skill.
2. Determine which nodes were affected and whether lock files are stale.
3. If tests need creation or update, invoke `/testing-[language]`.
4. Run tests and regenerate lock files for affected nodes.
5. Return lock status to calling skill.

## Current skills disposition

This table maps existing `spx` plugin skills to their disposition in the new `spec-tree` plugin.

| Current skill (`spx` plugin)            | Disposition in `spec-tree` plugin                                         |
| --------------------------------------- | ------------------------------------------------------------------------- |
| `understanding-durable-map`             | Absorbed into `understanding` as reference material                       |
| `understanding-assertion-decomposition` | Absorbed into `understanding` as reference material                       |
| `understanding-outcome-decomposition`   | Absorbed into `understanding`; rewritten for enabler/outcome model        |
| `managing-spx`                          | Split: templates/structure to `understanding`, workflows to action skills |
| `decomposing-prd-to-capabilities`       | Absorbed into `decomposing`; rewritten for tree decomposition             |
| `decomposing-capability-to-features`    | Absorbed into `decomposing`; rewritten for tree decomposition             |
| `decomposing-feature-to-stories`        | Absorbed into `decomposing`; rewritten for tree decomposition             |
| `writing-prd`                           | Absorbed into `authoring`; PRD becomes product spec (`.product.md`)       |
| `migrating-spec-to-spx`                 | Stays in `spx` plugin, moved to migration extension                       |
| `understanding-spx` (current)           | Rewritten as `understanding` foundation skill                             |
