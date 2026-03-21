---
name: contextualizing
description: >-
  ALWAYS invoke this skill when asking about status, progress, or what exists in the spec tree.
  NEVER work on any part of the spec tree without loading context through this skill first.
allowed-tools: Read, Glob, Grep
---

<!-- PLACEHOLDER: Full implementation in Phase 2 -->

<objective>

Walk the Spec Tree from product root to target node, deterministically collecting context: ancestor specs along the path, lower-index siblings' specs at each directory level. Emit `<SPEC_TREE_CONTEXT target="...">` marker with the collected context manifest.

</objective>
