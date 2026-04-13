# Contributing to SIA

Thank you for your interest in contributing to Schema Intelligence Annotations. SIA is an open specification and we welcome contributions of all kinds.

## Ways to Contribute

### Vocabulary Proposals

Propose new `x-sia-role` values for specific domains. Open an issue with:

- **Role name** (lowercase English word or hyphenated compound)
- **Domain** (e.g., healthcare, finance, geospatial)
- **Definition** (one sentence explaining what the role means)
- **Example properties** (3–5 typical property names that would use this role)
- **Justification** (why the existing standard roles don't cover this)

### Examples

Submit real-world SIA schemas from different industries. Place them in `examples/` with:

- A descriptive filename: `{domain}-{usecase}.sia.json`
- At least two types with relationships
- At least one `x-maps-to` mapping to a public standard
- All three SIA annotations used where appropriate

### Tooling

Build validators, converters, IDE plugins, or integrations. We maintain a list of community tools in the README.

### Platform Extensions

Define an `x-{platform}-*` namespace for your tool or platform. Document it in `docs/platform-extensions.md` with:

- Namespace prefix (e.g., `x-myplatform-`)
- List of properties with types and purposes
- When to include / omit them
- Round-trip fidelity expectations

## Process

1. **Open an issue** describing what you'd like to contribute
2. **Discuss** with maintainers and community
3. **Fork** the repository
4. **Make changes** in a feature branch
5. **Submit a pull request** referencing the issue
6. **Review** — maintainers will review within one week

## Specification Changes

Changes to the core vocabulary (`x-sia-role`, `x-sia-priority`, `x-sia-instruction`, `x-maps-to`) require:

- An issue with the `spec-change` label
- Discussion period of at least 14 days
- Consensus among maintainers
- Updated spec document, examples, and tests

## Style Guide

- Use **sentence case** in documentation (not Title Case)
- Code examples use **2-space indentation**
- JSON examples must be valid JSON (parseable by `JSON.parse`)
- YAML examples must round-trip to identical JSON
- Property names use **lowercase with hyphens** for SIA extensions

## Code of Conduct

Be respectful, constructive, and inclusive. We follow the [Contributor Covenant](https://www.contributor-covenant.org/version/2/1/code_of_conduct/).

## License

By contributing, you agree that your contributions will be licensed under the [Apache License 2.0](LICENSE).
