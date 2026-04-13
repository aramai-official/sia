# Schema Intelligence Annotations (SIA)

**Make schemas intelligent for AI agents, interoperable across standards, and portable across platforms.**

SIA is a vocabulary of `x-` vendor extensions for [JSON Schema](https://json-schema.org) that adds AI operational metadata, cross-standard mappings, and platform-specific round-trip data — without inventing a new format.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-green.svg)](LICENSE)
[![JSON Schema](https://img.shields.io/badge/Built%20on-JSON%20Schema-blue.svg)](https://json-schema.org)
[![Spec Version](https://img.shields.io/badge/Spec-v1.0-purple.svg)](https://www.schematica.io/sia)

---

## Why SIA?

Every major LLM uses JSON Schema for function calling and structured output. But JSON Schema alone doesn't tell an AI agent *what a field means*, *how important it is*, or *how it maps to external standards*. SIA fills those gaps with three self-describing extensions — no new syntax, no new parser, no new format to learn.

```json
"title": {
  "type": "string",
  "maxLength": 200,

  "x-sia-role": "identifier",
  "x-sia-priority": 1,
  "x-sia-instruction": "Main heading. Always include in any summary.",

  "x-maps-to": {
    "schema.org": "https://schema.org/headline",
    "Dublin Core": "http://purl.org/dc/elements/1.1/title"
  }
}
```

An LLM reading this knows: *title* identifies the entity, it should never be dropped, and it maps to `schema:headline` in schema.org. No training needed — the property names are plain English.

## Core Vocabulary

SIA adds exactly **four** extension properties. That's the entire spec.

| Property | Type | Purpose |
|---|---|---|
| `x-sia-role` | string | Semantic role: `"identifier"`, `"content"`, `"relationship"`, `"temporal"`, etc. |
| `x-sia-priority` | integer 1–5 | Truncation priority. 1 = always keep, 5 = drop first. |
| `x-sia-instruction` | string | Natural language instruction for AI. Anti-hallucination. |
| `x-maps-to` | object | Cross-standard mappings: `{"schema.org": "...", "FHIR": "..."}` |

### What SIA does NOT do

SIA never duplicates what JSON Schema already handles. Types, required fields, `$ref` relations, `enum` taxonomies, `allOf` inheritance, `minLength`, `pattern` — all native. SIA only adds what's missing: AI intelligence and cross-standard interoperability.

## Design Principles

1. **Never reinvent the screw.** If JSON Schema can express it, use it natively. Never create an `x-sia-*` version of something that already exists.
2. **Self-describing names.** An LLM that has never seen SIA should infer the meaning from the property name alone.
3. **Graceful ignorance.** Any JSON Schema tool processes SIA-annotated schemas without error. Standard validators ignore `x-` extensions by design.
4. **Layered enrichment.** Plain JSON Schema is valid SIA. Each layer is optional and additive.
5. **One file, multiple consumers.** The same file serves humans, AI agents, validators, and import tools.

## Architecture: Four Layers

```
┌─────────────────────────────────────────────┐
│  Layer 4: x-{platform}-*                    │  Platform-specific round-trip
│  Layer 3: x-maps-to                         │  Cross-standard mappings
│  Layer 2: x-sia-*                           │  AI intelligence
│  Layer 1: JSON Schema                       │  Structure & validation
└─────────────────────────────────────────────┘
```

Each layer is independently optional. Layer 1 is standard JSON Schema. Layers 2–3 are defined by this spec. Layer 4 is for any platform to define its own namespace (e.g., `x-cm-*` for a data modeling tool, `x-sanity-*` for a CMS).

## Quick Start

### 1. Start with a standard JSON Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "BlogPost",
  "type": "object",
  "properties": {
    "title": { "type": "string", "maxLength": 200 },
    "body": { "type": "string" },
    "author": { "$ref": "#/$defs/Author" },
    "category": { "enum": ["tech", "science", "culture"] }
  },
  "required": ["title", "body", "author"]
}
```

### 2. Add AI intelligence (Layer 2)

```json
"title": {
  "type": "string",
  "maxLength": 200,
  "x-sia-role": "identifier",
  "x-sia-priority": 1,
  "x-sia-instruction": "Main heading. Always include."
}
```

### 3. Add cross-standard mappings (Layer 3)

```json
"title": {
  "type": "string",
  "x-sia-role": "identifier",
  "x-sia-priority": 1,
  "x-maps-to": {
    "schema.org": "https://schema.org/headline",
    "Dublin Core": "http://purl.org/dc/elements/1.1/title",
    "Open Graph": "og:title"
  }
}
```

### 4. Optionally add platform extensions (Layer 4)

```json
"author": {
  "$ref": "#/$defs/Author",
  "x-sia-role": "relationship",
  "x-sia-priority": 2,
  "x-cm-relation": "domainIncludes",
  "x-cm-order": 4
}
```

## x-sia-role Values

| Role | Meaning | Example properties |
|---|---|---|
| `"identifier"` | Primary name, ID, title | name, title, slug, sku |
| `"descriptive"` | Summary, abstract, explanation | summary, bio, description |
| `"content"` | Main body, payload | body, text, html |
| `"relationship"` | Reference to another entity | author, parent (via `$ref`) |
| `"classification"` | Category, tag, grouping | category, tags, enum fields |
| `"temporal"` | Date, time, duration | createdAt, publishedDate |
| `"contact"` | Email, phone, address | email, phone, address |
| `"status"` | State, flag, lifecycle | status, active, published |
| `"metric"` | Numeric measurement | price, rating, count |
| `"governance"` | Audit, ownership | createdBy, accessLevel |

The vocabulary is extensible. Domain-specific roles (e.g., `"clinical"`, `"financial"`) can be added as lowercase English words.

## x-sia-priority Values

| Value | Label | Meaning |
|---|---|---|
| 1 | Essential | Always include, even in one-line summaries |
| 2 | Important | Include in standard context |
| 3 | Useful | Include in full display, drop under pressure |
| 4 | Supplementary | Include only with generous token budget |
| 5 | Background | Drop first, rarely needed by AI |

### Progressive truncation

SIA-aware tools can use `maxPriority` to progressively strip annotations:

```
maxPriority=5  →  Full: all annotations
maxPriority=3  →  Normal: priority 1–3 only
maxPriority=2  →  Lean: priority 1–2 only
maxPriority=1  →  Minimal: priority 1 only
maxPriority=0  →  Structure: no SIA annotations
```

This is more intelligent than format compression — you selectively remove the least important metadata while keeping the most important.

## YAML Serialisation

SIA schemas can be serialised as YAML. Benchmarks show YAML achieves 12–18% higher LLM comprehension accuracy for nested data, with ~18% fewer tokens.

```yaml
$defs:
  BlogPost:
    type: object
    x-sia-instruction: >-
      A blog post. Title and author always needed.
    x-maps-to:
      schema.org: https://schema.org/BlogPosting
    properties:
      title:
        type: string
        maxLength: 200
        x-sia-role: identifier
        x-sia-priority: 1
        x-maps-to:
          schema.org: https://schema.org/headline
          Dublin Core: http://purl.org/dc/elements/1.1/title
      author:
        $ref: "#/$defs/Author"
        x-sia-role: relationship
        x-sia-priority: 2
        x-sia-instruction: Use author name in summaries, not ID.
    required: [title, body, author]
```

**Recommendation:** Use YAML for AI consumption (compact, better comprehension). Use JSON for code repositories and API tooling (universal compatibility).

## Complete Example

See [`examples/blog-content-model.sia.json`](examples/blog-content-model.sia.json) for a full schema with two types, inheritance, a taxonomy, and mappings to schema.org and Dublin Core.

## Repository Structure

```
sia-spec/
├── README.md                          # This file
├── LICENSE                            # Apache 2.0
├── CONTRIBUTING.md                    # How to contribute
├── CHANGELOG.md                       # Version history
│
├── spec/
│   └── sia-v1.md                      # Full specification document
│
├── schema/
│   ├── sia-meta.schema.json           # JSON Schema for validating SIA extensions
│   └── x-maps-to.schema.json          # JSON Schema for validating x-maps-to objects
│
├── examples/
│   ├── blog-content-model.sia.json    # Blog with posts and authors
│   ├── blog-content-model.sia.yaml    # Same example in YAML
│   ├── ecommerce.sia.json             # E-commerce product catalog
│   ├── healthcare-fhir.sia.json       # Healthcare with FHIR mappings
│   └── minimal.sia.json               # Smallest valid SIA schema
│
├── tools/
│   ├── validate.js                    # Node.js SIA validator
│   ├── strip.js                       # Strip SIA annotations by priority
│   ├── convert.js                     # JSON ↔ YAML converter
│   └── token-count.js                 # Token counter for SIA schemas
│
└── docs/
    ├── site/                          # Specification website source
    ├── roles.md                       # Extended role vocabulary guide
    ├── mappings.md                    # Cross-standard mapping guide
    └── platform-extensions.md         # Guide for defining x-{platform}-* extensions
```

## Validation

A SIA schema is valid if:

1. It is a valid JSON Schema (passes any standard JSON Schema validator)
2. All `x-sia-role` values are lowercase English words from the standard or extended vocabulary
3. All `x-sia-priority` values are integers between 1 and 5
4. All `x-sia-instruction` values are non-empty strings
5. All `x-maps-to` values are objects with string keys and string values
6. No `x-sia-*` property duplicates a native JSON Schema capability

```bash
# Validate a SIA schema
npx sia-validate my-schema.sia.json

# Strip annotations to a specific priority level
npx sia-strip my-schema.sia.json --max-priority 3

# Count tokens
npx sia-tokens my-schema.sia.json
```

## FAQ

### Is SIA a new schema language?

No. SIA is standard JSON Schema with `x-` vendor extensions. Every JSON Schema tool reads SIA schemas without modification. Every LLM that understands JSON Schema (which is all of them — it's the format they use for function calling) understands SIA schemas.

### Why not ShExC, CDDL, or another compact format?

We evaluated every viable alternative. JSON Schema won on the only metric that matters for AI: **comprehension accuracy**. LLMs are specifically fine-tuned on JSON Schema for tool use. Compact formats like ShExC save tokens but carry comprehension risk. SIA uses progressive truncation (`x-sia-priority`) for token efficiency instead of format compression.

### Why `x-` prefix and not a custom keyword?

The `x-` vendor extension mechanism is defined by the [OpenAPI Specification](https://swagger.io/specification/) and is the standard way to extend JSON Schema. LLMs have seen millions of OpenAPI specs with `x-` extensions. They know to read but not break on unrecognised `x-` properties. This is not a workaround — it's the designed extension point.

### Can I define my own platform extensions?

Yes. Any platform can define its own `x-{platform}-*` namespace. A data modeling tool might use `x-cm-*`, a CMS might use `x-sanity-*`, an ETL platform might use `x-dbt-*`. Layer 4 is deliberately open. See [`docs/platform-extensions.md`](docs/platform-extensions.md).

### How does x-maps-to relate to JSON-LD @context?

`x-maps-to` follows the same principle (mapping local names to URIs) but uses a simpler structure that doesn't require JSON-LD processing. An AI can read `x-maps-to` and generate a JSON-LD `@context` from it — but the mapping itself is plain JSON, readable without any linked data tooling.

### What about token efficiency?

SIA adds ~30% more tokens compared to bare JSON Schema. But `x-sia-priority` enables progressive truncation — strip lower-priority annotations to fit any budget. A schema at `maxPriority=1` is barely larger than plain JSON Schema, while carrying the most essential AI intelligence. YAML serialisation saves an additional ~18% tokens.

## Roadmap

- [x] Core vocabulary: `x-sia-role`, `x-sia-priority`, `x-sia-instruction`
- [x] Cross-standard mappings: `x-maps-to`
- [x] Platform extension pattern: `x-{platform}-*`
- [x] YAML serialisation support
- [x] Progressive truncation via `maxPriority`
- [ ] Governance annotations: `x-sia-claim-level`, `x-sia-gate`
- [ ] Validation tooling (npm package)
- [ ] VS Code extension for SIA authoring
- [ ] JSON-LD `@context` generation from `x-maps-to`
- [ ] Exemplar / sample data embedding pattern

## Contributing

SIA is an open specification. We welcome contributions of all kinds:

- **Vocabulary proposals** — new `x-sia-role` values for specific domains
- **Examples** — real-world SIA schemas from different industries
- **Tooling** — validators, converters, IDE plugins
- **Platform extensions** — `x-{platform}-*` namespace definitions
- **Documentation** — guides, tutorials, translations

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Community

SIA is powered by the [Schematica](https://www.schematica.io) community — an open ecosystem for schema-aware work.

- **Website:** [schematica.io/sia](https://www.schematica.io/sia)
- **GitHub:** [github.com/schematica/sia-spec](https://github.com/schematica/sia-spec)
- **Discussions:** [GitHub Discussions](https://github.com/schematica/sia-spec/discussions)

## License

This specification is licensed under [Apache License 2.0](LICENSE).

SIA is built on [JSON Schema](https://json-schema.org) and inspired by the [OpenAPI](https://www.openapis.org/) extension pattern.

---

<p align="center">
  <strong>SIA</strong> — Schema Intelligence Annotations<br>
  <em>One file. Three pillars. Zero reinvention.</em><br><br>
  Powered by <a href="https://www.schematica.io">Schematica</a>
</p>
