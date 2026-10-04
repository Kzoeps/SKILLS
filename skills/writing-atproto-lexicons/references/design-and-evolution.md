# Lexicon design, review, and evolution

Source: https://atproto.com/guides/lexicon-style-guide and https://atproto.com/specs/lexicon (retrieved 2026-07-24). Requirements below are labeled separately from recommendations.

## Naming and namespace design

Official style recommendations:

| Concept | Convention | Example |
| --- | --- | --- |
| Schema final names and fields | lower camel case | `getPost`, `createdAt` |
| API errors | upper camel case | `RecordNotFound` |
| Fixed strings and simple known values | kebab case | `pending-review` |
| Record schema | singular noun | `post`, `profile` |
| Query/procedure | verb + noun | `getPost`, `listLikes`, `putProfile` |
| Subscription | `subscribe` + plural noun | `subscribeLabels` |
| Permission set | unsettled; likely `auth` prefix | `authBasic` |

Common query verbs: `get`, `list`, `search` for full-text search, and `query` for flexible matching/filtering. Common procedure verbs: `create`, `update`, `delete`, `upsert`, and `put`.

Field names should use the NSID-name character set: ASCII alphanumeric, case-sensitive, and not starting with a digit. Do not use hyphens in field names. Preserving a field name from an external schema can justify an exception. Schema-defined names beginning with `$` are reserved and prohibited.

Avoid generic schema names that collide with common language concepts, such as `default` and `length`.

Group distinct features in the NSID hierarchy, such as `app.example.feed.*` and `app.example.graph.*`. Keep reusable definitions in a stable `*.defs` schema. Avoid names that make a fragment and nested NSID ambiguous, such as simultaneously creating `com.example.record#foo` and `com.example.record.foo`.

Use `.temp.` or `.unspecced.` in unstable, experimental, or non-interoperable endpoint namespaces. This is a recommendation, but it prevents accidental commitment to a published data contract.

## Documentation

Every primary `main` definition should have a description. This is a style requirement, not a Lexicon parser requirement.

For endpoints, state:

- whether authentication is required
- if authentication is optional, whether it personalizes the response

Describe fields whose referent is not obvious. `uri`, `cid`, `value`, `subject`, and `type` often need clarification: URI or CID of what, value in which units, and subject of which relationship?

Descriptions should state contracts, not repeat property names.

## Field modeling

### Strings

Use a semantic format when one exists. Record strings without a format should almost always have a maximum byte length.

Do not redundantly add custom length constraints to a formatted string merely to restate its format. If product or visual design imposes a human-facing limit, specify both:

- a grapheme limit for consistency across writing systems
- a UTF-8 byte limit for bounded storage

A byte-to-grapheme ratio between 10:1 and 20:1 is the official recommendation. Choose the actual limits from product needs rather than copying a universal constant.

Use blobs for large text, binary content, or larger structured data. Short strings and `bytes` are intended for constrained data.

### Value sets

Avoid `enum` unless the set is inherently closed forever. Removing enum values or allowing new values later conflicts with schema evolution.

Prefer:

```json
{
  "type": "string",
  "knownValues": ["draft", "published", "com.example.defs#archived"]
}
```

`knownValues` are suggestions, not validation constraints. Tokens are useful when meanings are subjective, externally extensible, or likely to grow.

### References

Use `com.atproto.repo.strongRef` for a version-specific reference to another record. Its string `uri` plus string `cid` shape is the usual application-level representation.

Use a plain DID-formatted string for another account in stored records. Handles are mutable and therefore inappropriate as persistent account references. Endpoint inputs can use `at-identifier` to save callers from resolving handles themselves.

### Arrays and unions

Prefer an object element over an atomic element when future context may be needed:

```json
{
  "type": "array",
  "items": {
    "type": "ref",
    "ref": "#accountItem"
  }
}
```

where `accountItem` initially has one `account` property. A later optional role or timestamp can then be added without replacing the array's item type.

Keep unions open unless the set is intrinsically closed. Open unions allow both the authority and third parties to introduce variants. Consumers must tolerate `$type` values they do not recognize.

### Optional booleans

Name optional booleans so omission and `false` produce the ordinary behavior. Prefer `includeArchived` or `excludeDefaultItems` over a parameter that needs `default: true` to express the common path.

## Evolution analysis

The compatibility invariant has two directions:

1. Every old valid datum remains valid under the new schema.
2. Every new valid datum remains valid under the old schema.

Because repository records are distributed, schema authors cannot reliably migrate every old record or update every old client.

### Usually compatible

- Add an optional object field.
- Add a variant to an open union.
- Add a `knownValues` suggestion while preserving the field as an open string.
- Keep an unused field, unchanged, and mark it deprecated in its description.

Before declaring compatibility, still consider whether newly generated data remains acceptable to older validators. For example, adding a new `knownValues` entry is safe because the set is open; adding a new enum value is not safe for an old closed validator.

### Breaking or evolution-sensitive

- Add a required field.
- Remove a required field.
- Rename any field.
- Change a field's type.
- Remove a union variant.
- Convert an open union to closed.
- Narrow lengths, numeric ranges, accepted MIME types, or other constraints such that old valid data fails.
- Broaden a closed constraint such that new data fails old validators, including adding enum values.
- Replace an atomic array item with an object, even if the new object looks semantically equivalent.

Optional fields generally should not be removed either: preserving them and marking them deprecated prevents old clients from having their still-valid data stripped or reinterpreted.

For a breaking change to a released schema, create a new NSID. The current convention appends `V2`, then `V3`, to the original schema name. Keep the old schema available.

Public adoption or implementation by a third party is enough to treat a Lexicon as released, even if the original author did not announce a stable release.

## Review checklist

### Language validity

- [ ] `lexicon` is exactly `1`; `id` is a valid NSID; `defs` is non-empty.
- [ ] There is at most one primary definition and it is named `main`.
- [ ] Every schema node uses a type valid in that context.
- [ ] Named defs do not use forbidden scoped/meta types.
- [ ] `required` and `nullable` names exist in `properties`; params do not use `nullable`.
- [ ] Refs resolve and do not target tokens, refs, or unions.
- [ ] Union refs resolve to object or record definitions; closed unions are non-empty.
- [ ] Permission-set resources are in the set's own NSID group or descendant groups; `rpc` permissions do not combine `inheritAud: true` with `aud`, and inherited audiences are provided by the invoking `include` scope.
- [ ] Subscription messages use a union of refs.
- [ ] String `const` and `default` do not coexist.
- [ ] Blob MIME globs are valid.
- [ ] No schema-defined property begins with `$`.

### Style and interoperability

- [ ] Names use the appropriate casing and noun/verb convention.
- [ ] The main definition and ambiguous fields have useful descriptions.
- [ ] Endpoint descriptions cover authentication and personalization.
- [ ] Endpoint output and encoding are explicit.
- [ ] Account endpoint params use `at-identifier`; stored account references use `did`.
- [ ] Record strings have formats or justified limits.
- [ ] Human-facing limits include grapheme and byte bounds.
- [ ] Closed enums/unions have a documented reason.
- [ ] Cross-schema concepts use stable defs; record references use strong refs when version identity matters.
- [ ] Optional booleans default naturally to false.
- [ ] Pagination, subscription sequencing, and hydrated views follow their patterns when applicable.

### Evolution

- [ ] New fields are optional unless this is a new, unpublished schema and functionality truly requires them.
- [ ] Existing fields retain names, types, and valid ranges.
- [ ] Required fields are not removed or made optional.
- [ ] Existing union variants remain present.
- [ ] New variants are added only to open unions.
- [ ] Deprecated fields remain representable.
- [ ] Breaking changes use a new NSID rather than silently replacing a published schema.

## Review wording

Cite a **specification rule** for invalid schemas and a **style-guide recommendation** for design warnings. Do not inflate recommendations into parser errors. If validity depends on another Lexicon that is not present, mark the finding as uncertain until the target definition is inspected.
