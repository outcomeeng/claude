# Skill Structure - Migration: spx-legacy to spec-tree

## Purpose

This document defines the migration path from spx-legacy trees (capability/feature/story) to spec-tree format (enabler/outcome). Products using the `spx-legacy` plugin can migrate incrementally to `spec-tree` without a big-bang rewrite.

Products that do not require migration should use only:

- `methodology/skills/skill-structure.md`

## What changes between systems

### Shared (no migration needed)

Both systems share core principles. These require no migration:

- Durable map philosophy (specs are permanent, never closed)
- Test-derived status (no status fields in specs)
- BSP numbering with sparse integers in [10, 99]
- Atemporal voice (no temporal markers in specs)
- ADR/PDR interleaving at any scope level
- Test co-location in `tests/` subdirectories
- Test level determined by evidence needed, not by hierarchy level

### Structural changes

| Aspect           | spx-legacy                                   | spec-tree                          |
| ---------------- | -------------------------------------------- | ---------------------------------- |
| Node types       | 3 fixed: `.capability`, `.feature`, `.story` | 2 flexible: `.enabler`, `.outcome` |
| Nesting depth    | Fixed 3 levels                               | Arbitrary depth                    |
| Spec file naming | `{slug}.{type}.md`                           | `{slug}.md`                        |
| Product file     | `{product}.prd.md`                           | `{product}.product.md`             |

### Content changes

| Aspect          | spx-legacy                           | spec-tree                                                                     |
| --------------- | ------------------------------------ | ----------------------------------------------------------------------------- |
| Spec header     | `## Purpose` + `## Requirements`     | Enabler: `## Enables` / Outcome: hypothesis + `## Assertions`                 |
| Outcome format  | `## Outcomes` with numbered sections | `## Assertions` with typed assertion lists                                    |
| Test Strategy   | Explicit `## Test Strategy` table    | No separate section; assertions carry test links inline                       |
| Assertion types | Implicit                             | Five explicit types: Scenarios, Mappings, Conformance, Properties, Compliance |

## Migration strategy

### Phase 1: Structural rename (scriptable)

Mechanical transformations that preserve content. Reversible via git.

1. **Classify each node** as enabler or outcome:
   - Infrastructure nodes (test harnesses, shared utilities, build tooling) → `.enabler`
   - Everything else → `.outcome`
   - When unsure: if removing the node would only break other nodes (not users), it is an enabler

2. **Rename directories**: `.capability` / `.feature` / `.story` → `.enabler` or `.outcome`

3. **Rename spec files**: `{slug}.capability.md` / `{slug}.feature.md` / `{slug}.story.md` → `{slug}.md`

4. **Rename product file** (if present): `{product}.prd.md` → `{product}.product.md`

5. **Validate**: Run `/aligning` on the migrated tree. It will report `.capability`/`.feature`/`.story` suffixes as unrecognized — these are the nodes you missed.

### Phase 2: Content rewrite (incremental, per-node)

Content rewriting requires domain understanding and cannot be fully automated. Do this incrementally as you work on each node, not all at once.

For each node:

1. **Enablers**: Replace `## Purpose` + `## Requirements` with `## Enables` section stating what this enabler provides to its dependents

2. **Outcomes**: Rewrite to hypothesis format:
   - Replace `## Purpose` with hypothesis: `WE BELIEVE THAT {output} WILL {outcome} CONTRIBUTING TO {impact}`
   - Replace `## Outcomes` (numbered sections) with `## Assertions` using typed assertion lists
   - Move Test Strategy table content into inline assertion `[test]` links
   - Classify each assertion under the appropriate heading (Scenarios, Mappings, Conformance, Properties, Compliance)

3. **Remove sections that have no spec-tree equivalent**:
   - `## Success Metric` (absorbed into hypothesis outcome/impact)
   - `## Test Strategy` (absorbed into assertion test links)
   - `## Analysis` from stories (implementation detail belongs in code, not specs)

### Phase 3: Flatten unnecessary nesting (optional)

spx-legacy enforces exactly 3 levels. After migration, review whether the depth is justified:

- A story with a single assertion may not need its own directory — its assertion could live in the parent outcome
- A capability with a single feature may collapse into one outcome
- Conversely, a deeply nested story may warrant its own child outcomes

This is refactoring, not migration. Use `/refactoring` when ready.

## Node type classification guide

### Common enabler candidates

| spx-legacy pattern                  | Why enabler                           |
| ----------------------------------- | ------------------------------------- |
| `NN-test-infrastructure.capability` | Pure infrastructure for other nodes   |
| `NN-infrastructure.capability`      | Shared tooling, no direct user value  |
| `NN-transient.capability`           | Staging/bootstrap, serves other nodes |
| Build pipeline features             | Enables deployment, not user-facing   |
| Test harness features               | Enables testing, not user-facing      |

### Common outcome candidates

| spx-legacy pattern         | Why outcome                               |
| -------------------------- | ----------------------------------------- |
| Most capabilities          | Deliver user-facing product value         |
| Most features              | Deliver testable user-facing behavior     |
| Most stories               | Atomic testable behavior                  |
| Documentation capabilities | Deliver value to users (docs are product) |

### Decision heuristic

Ask: "If every node that depends on this were removed, would this node still have value to users?"

- **Yes** → outcome (it has independent user value)
- **No** → enabler (it exists only to serve dependents)

## What NOT to migrate

- **ADR/PDR files**: Format is compatible. Only rename if content needs atemporal voice cleanup.
- **Test files**: Same co-location pattern, same naming convention. No changes needed.
- **BSP numbers**: Same system. No renumbering needed.
- **`spx/CLAUDE.md`**: Update to reference spec-tree conventions, but this is project configuration, not migration.
