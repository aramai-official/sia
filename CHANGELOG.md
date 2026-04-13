# Changelog

All notable changes to the SIA specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-04-13

### Added

- Core vocabulary: `x-sia-role`, `x-sia-priority`, `x-sia-instruction`
- Cross-standard mappings: `x-maps-to`
- Platform extension pattern: `x-{platform}-*`
- Standard role vocabulary: identifier, descriptive, content, relationship, classification, temporal, contact, status, metric, governance
- Priority scale: 1 (essential) through 5 (background)
- Progressive truncation via `maxPriority` parameter
- YAML serialisation support
- Complete specification document
- Blog content model example (JSON and YAML)
- Repository structure and contribution guidelines

### Design decisions

- JSON Schema chosen as base format for LLM comprehension accuracy (fine-tuned for function calling across all major LLMs)
- `x-` vendor extension mechanism chosen for compatibility with OpenAPI ecosystem
- Self-describing English property names chosen over abbreviated codes
- Four-layer architecture chosen for clean separation of concerns
- Platform extensions use open `x-{name}-*` namespace pattern (not locked to any platform)
