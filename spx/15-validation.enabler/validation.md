# Validation

## Enables

Pre-commit validation infrastructure that catches invalid plugin artifacts before they enter the repository. The marketplace's existing `validate-marketplace` and `validate-plugins` hooks validate `plugin.json` manifests. This enabler extends validation to SKILL.md frontmatter, ensuring skill files conform to the fields the Claude Code CLI actually accepts.

## Field Source

The set of valid SKILL.md frontmatter fields is extracted from the installed Claude Code CLI binary. The binary embeds JavaScript source that parses frontmatter via property access patterns (`var["field-name"]`, `var.field`). Extracting these from the binary ensures the validation stays current as Claude Code evolves — no hardcoded list to maintain.

The Agent Skills open standard fields (`name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`) serve as a minimum floor. The binary-extracted fields serve as the maximum ceiling. Any frontmatter key outside this union is invalid.

## Assertions

### Scenarios

- Given a SKILL.md with only standard Agent Skills fields, when validated, then no errors are reported ([test](tests/test_validation_unit.py))
- Given a SKILL.md with Claude Code extension fields (`model`, `effort`, `context`, `agent`, `hooks`), when validated, then no errors are reported ([test](tests/test_validation_unit.py))
- Given a SKILL.md with an unknown field (`foo-bar`), when validated, then an error is reported naming the invalid field ([test](tests/test_validation_unit.py))
- Given a SKILL.md with no frontmatter, when validated, then no errors are reported ([test](tests/test_validation_unit.py))
- Given a file that is not named SKILL.md, when passed to the validator, then it is skipped ([test](tests/test_validation_unit.py))

### Properties

- Field extraction is deterministic: the same binary always produces the same field set ([test](tests/test_validation_unit.py))
- The standard fields are always a subset of the valid fields, regardless of binary extraction success or failure ([test](tests/test_validation_unit.py))

### Compliance

- ALWAYS: fall back to the Agent Skills standard fields when binary extraction fails — never fail open with an empty set, never fail closed by rejecting all fields
- NEVER: hardcode Claude Code-specific fields — they are derived from the binary at runtime
