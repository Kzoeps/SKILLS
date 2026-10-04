---
name: writing-atproto-lexicons
description: Design, write, review, and evolve AT Protocol Lexicon v1 schemas for records, XRPC queries and procedures, subscriptions, permission sets, and reusable defs. Use whenever creating or editing Lexicon JSON, choosing NSIDs or field shapes, designing refs/unions/string limits, checking schema evolution, or reviewing whether an atproto schema follows the Lexicon specification and style guide—even if the user only says “ATProto schema” or provides a lexicons directory. Do not use merely to consume generated Lexicon types in application code unless schema authoring or review is involved.
---

# Writing AT Protocol Lexicons

Design Lexicon schemas that are valid today and can evolve without invalidating distributed records or older clients.

## Canonical sources

Treat these as authoritative when this skill conflicts with memory or examples:

- Lexicon v1 specification: https://atproto.com/specs/lexicon
- Lexicon style guide: https://atproto.com/guides/lexicon-style-guide

The bundled references summarize those documents; re-check the live sources when the user asks about newly introduced language features, publication, permissions, or a disputed rule.

## Select the task

- **Create or edit a schema:** follow the authoring workflow, then the validation workflow.
- **Review schemas:** do not rewrite first. Follow the review workflow and report findings with exact paths.
- **Assess an evolution:** compare old and new definitions before editing. Classify compatibility and recommend a new NSID for breaking changes.
- **Explain Lexicon:** answer from [references/specification.md](references/specification.md), distinguishing language requirements from style recommendations.

Read [references/design-and-evolution.md](references/design-and-evolution.md) for every authoring or review task. Read [references/patterns.md](references/patterns.md) only for the primary type or design pattern involved.

## Authoring workflow

### 1. Inspect local conventions

Before writing:

1. Read repository instructions and the existing Lexicon directory.
2. Identify generation or validation tooling from package scripts, build files, and nearby schemas.
3. Match established file layout and formatting unless either violates the specification.
4. Search existing schemas before adding a new def; prefer established shared definitions and protocol types such as `com.atproto.repo.strongRef`.

Do not add dependencies or invent a new generator when the repository already has a supported path.

### 2. Establish the contract

Determine, from the request or surrounding code:

```ts
type LexiconIntent = {
  nsid: string
  primaryType: "record" | "query" | "procedure" | "subscription" | "permission-set" | "defs"
  stability: "experimental" | "published" | "unknown"
  authentication?: "required" | "optional" | "none"
  personalizedResponse?: boolean
  recordKey?: string
  requiredInvariants: string[]
  reusableConcepts: string[]
}
```

Ask only for information that cannot safely be inferred. If the namespace is experimental or interoperability is not intended, recommend a `.temp.` or `.unspecced.` NSID segment.

### 3. Design for extension

Work backward from the minimum interoperable contract:

- Make a field required only when the record or endpoint cannot function without it.
- Prefer optional additions and open unions.
- Prefer objects in arrays when an element may need context later.
- Use `knownValues` or token references instead of closed string `enum` unless the value set is permanently closed by definition.
- Put cross-schema reusable definitions in a dedicated `*.defs` Lexicon.
- Use DIDs for account references stored in records; use `at-identifier` for endpoint account parameters that should accept either a handle or DID.
- Use `com.atproto.repo.strongRef` for versioned record references unless the domain specifically needs an unversioned reference.
- Give record strings an appropriate `format`, or usually a byte limit; when imposing a human-facing limit, use both grapheme and byte limits.
- Use blobs for large text, binary data, or structured payloads that do not belong in constrained record fields.

Explain consequential modeling choices in the response; do not add narrative comments to JSON.

### 4. Write valid Lexicon v1 JSON

Start every file with:

```json
{
  "lexicon": 1,
  "id": "com.example.group.schemaName",
  "defs": {}
}
```

Then enforce the structural rules in [references/specification.md](references/specification.md). In particular:

- A file has at least one definition and at most one primary definition.
- Name a primary definition `main`.
- Do not declare `ref`, `union`, `unknown`, or scoped `params`/`permission` types as named defs.
- Do not schema-define properties beginning with `$`; `$type` is a protocol discriminator in encoded data, not a property to add to a record object's `properties`.
- Reference a `main` definition by NSID without `#main`.
- For permission sets, ensure every referenced `repo` collection or `rpc` endpoint is in the set's own NSID group or a descendant group. For `rpc` permissions, never combine `inheritAud: true` with `aud`; an inherited audience also requires an `aud` on the invoking `include` scope.
- Ensure every main schema has a description. Endpoint descriptions state authentication requirements and optional personalization behavior.
- Every query and procedure has an output with an encoding, including endpoints with no meaningful response body.

### 5. Validate in layers

Run checks in this order so errors are actionable:

1. **JSON syntax:** parse every touched file.
2. **Lexicon language validity:** use the repository's installed validator or generator when available.
3. **Cross-reference integrity:** verify local fragments, global NSIDs, union targets, `required`/`nullable` property names, and permission-set resource namespace and audience constraints.
4. **Style and evolvability:** apply [references/design-and-evolution.md](references/design-and-evolution.md).
5. **Compatibility:** if replacing an existing schema, compare old and new versions field by field.
6. **Repository checks:** run the narrowest relevant generation, tests, typecheck, or lint commands.

Never claim formal validation from visual inspection alone. State when no Lexicon validator was available.

## Review workflow

Review before modifying. Report only current, actionable findings:

```md
## Findings

### Error — <short title>
- Location: `path/to/schema.json` → `defs.main...`
- Rule: <specification or evolution rule>
- Impact: <invalid schema, invalid data, or compatibility break>
- Fix: <specific correction>

### Warning — <short title>
- Location: ...
- Guideline: <style/evolvability recommendation>
- Risk: ...
- Fix: ...
```

Use these levels:

- **Error:** violates Lexicon v1, contains an invalid reference, or breaks compatibility for a published schema.
- **Warning:** violates the official style guide or creates a concrete evolution/interoperability risk.
- **Suggestion:** optional improvement with no current correctness or compatibility impact.

If there are no findings, say so and list the checks performed. Do not present preferences as specification errors.

## Evolution workflow

Given `old` and `new`, classify every change:

```ts
type EvolutionAssessment = {
  compatible: string[]
  breaking: Array<{
    path: string
    rule: string
    migration: "retain-old-shape" | "new-nsid-v2" | "experimental-only"
  }>
  uncertain: string[]
}
```

Generally compatible:

- adding an optional field
- adding a ref to an open union
- retaining a deprecated field without changing its type or requirement

Breaking or unsafe for a published Lexicon:

- adding a required field
- removing a required field
- renaming a field
- changing a field's type
- removing a union variant
- narrowing previously valid constraints or closing an open value space in a way old/new validators disagree on

The governing invariant is bidirectional: old data remains valid under the new schema, and new data remains valid under the old schema. When that cannot hold, preserve the old Lexicon and create a new schema, conventionally with `V2`, `V3`, and so on appended to the schema name.

## Output expectations

When authoring:

1. Write only requested schema files and directly affected tests/docs/generated artifacts.
2. Report the NSIDs and primary types created or changed.
3. Summarize required-field, union, reference, and limit decisions.
4. List exact validation commands and results.
5. Call out compatibility assumptions and unavailable validators.

When reviewing, do not edit unless the user asks for fixes. When a requested change would break a published schema, stop before rewriting it and present the compatible alternatives.

Do not publish Lexicons, modify DNS, write `com.atproto.lexicon.schema` records, or call a PDS unless the user explicitly asks and confirms the external action.
