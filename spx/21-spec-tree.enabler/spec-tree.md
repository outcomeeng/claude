# Spec Tree

PROVIDES the spec-driven development methodology — context loading, spec authoring, testing, implementation, and commit workflows
SO THAT all language-specific and craft plugins
CAN operate within a structured, spec-first framework with deterministic context

## Assertions

### Compliance

- ALWAYS: load complete spec-tree context before any implementation work ([review])
- ALWAYS: use atemporal voice in all specs — specs are permanent truth ([review])
- NEVER: proceed with partial context — abort if any required document is missing ([review])
