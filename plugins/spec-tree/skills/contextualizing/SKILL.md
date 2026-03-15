---
name: contextualizing
description: |
  When the user asks about status, progress, what exists, what's next, or where things stand, invoke this.
  When the user asks to work on any part of the project, invoke this first to load context.
  When the user asks "what do we have", "show me", "where are we", invoke this.
allowed-tools: Read, Glob, Grep
---

<!-- PLACEHOLDER: Full implementation in Phase 2 -->

<objective>

Walk the Spec Tree from product root to target node, deterministically collecting context: ancestor specs along the path, lower-index siblings' specs at each directory level. Emit `<SPEC_TREE_CONTEXT target="...">` marker with the collected context manifest.

</objective>
