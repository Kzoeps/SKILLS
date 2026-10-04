# Lexicon v1 specification reference

Source: https://atproto.com/specs/lexicon (retrieved 2026-07-24). The live specification is authoritative.

## File and definition rules

A Lexicon file is a JSON object with:

- `lexicon`: required integer, exactly `1`
- `id`: required NSID
- `description`: optional file overview
- `defs`: required non-empty map of named definitions

A file may contain at most one primary definition. Primary definitions are `record`, `query`, `procedure`, `subscription`, and `permission-set`; name the primary definition `main`.

A `*.defs` file should generally omit `main`. References use:

- `com.example.schema` for its `main` definition
- `com.example.defs#someObject` for another named definition
- `#someObject` for a definition in the same file

Never use `#main` in encoded `$type` values. A record's `$type` is its NSID.

Allowed named definition types include primary types plus concrete/container definitions such as `object`, `array`, `string`, `integer`, `boolean`, `bytes`, `cid-link`, `blob`, and `token`. Do not declare `params`, `permission`, `ref`, `union`, or `unknown` as named top-level defs.

## Primary definitions

### Record

```json
{
  "type": "record",
  "description": "...",
  "key": "tid",
  "record": {
    "type": "object",
    "properties": {},
    "required": []
  }
}
```

- `key` is required.
- `record` is required and must be an `object` schema.
- Encoded record data always includes `$type`, but do not define `$type` in `record.properties`.

### Query and procedure

- Query maps to XRPC HTTP GET; procedure maps to HTTP POST.
- `parameters`, when present, has type `params`.
- A procedure may define `input`; a query may not.
- `input` and `output` require `encoding`; their optional `schema` is an `object`, `ref`, or `union` of refs.
- `errors` entries require a whitespace-free `name` and may have a description.

An empty JSON response schema is valid and preferred over omitting endpoint output:

```json
"output": {
  "encoding": "application/json",
  "schema": { "type": "object", "properties": {} }
}
```

### Subscription

- `parameters` follows the endpoint params rules.
- `message.schema` is required and must be a `union` of refs, not an object.
- `errors` follows endpoint error rules.

At the top level of subscription messages, union variants are the exception to the general requirement for a `$type` discriminator.

### Permission set

- `permissions` is a required array of scoped `permission` definitions.
- Optional display fields are `title`, `title:lang`, `detail`, and `detail:lang`.
- Current Lexicon permission resources are `repo` and `rpc`.
- `repo.collection` is a required non-empty unique NSID array; wildcard is unsupported. Optional `action` contains only `create`, `update`, and/or `delete`.
- `rpc.lxm` is a required non-empty endpoint NSID array; wildcard is unsupported there. `aud` is required unless `inheritAud` is true. `aud` may be `*`, but `aud` and `lxm` cannot both be wildcard.
- A permission set may reference resources in its own NSID group or descendant groups, but not parent or sibling groups. Apply this to every `repo.collection` and `rpc.lxm` entry.
- For `rpc`, `inheritAud: true` and `aud` are mutually exclusive. An inherited audience must be supplied on the invoking `include` scope; without it, that permission is invalid and must be ignored.
- Unsupported resource types or permission fields must be ignored by access-control implementations, because future fields may attenuate access; never treat an unknown field as permission to grant more access.

Permission semantics may evolve. Re-read the live Lexicon and permission specifications before designing security-sensitive permission sets.

## Field types

### Object

- `properties`: required map of field schemas
- `required`: optional array naming properties that must be present
- `nullable`: optional array naming properties allowed to be `null`

Omitted, `null`, and false-y values are semantically distinct. Every name in `required` and `nullable` should identify a declared property. Schema-defined field names must not begin with `$`.

### Params

`params` is valid only as an endpoint's `parameters`. Its properties may be `boolean`, `integer`, `string`, or arrays of one of those types. It has `required` but no `nullable`.

### String

Supports:

- `format`
- UTF-8 byte constraints: `minLength`, `maxLength`
- Unicode grapheme constraints: `minGraphemes`, `maxGraphemes`
- open suggestions: `knownValues`
- closed values: `enum`
- `default` or `const`, but not both

String formats are:

- `at-identifier`
- `at-uri`
- `cid`
- `datetime`
- `did`
- `handle`
- `nsid`
- `tid`
- `record-key`
- `uri`
- `language`

`datetime` requires full date and time through whole seconds plus timezone. Uppercase `T` is required; uppercase `Z` is preferred. At least millisecond precision is recommended. Preserve the source string when hash-stable round trips matter, because native datetime conversions can lose precision or trailing zeros.

`at-identifier` accepts a DID or handle and is mainly appropriate for endpoint parameters. `did` is appropriate for persistent account references in records.

### Integer and boolean

Both support `default` and `const`; integer also supports `minimum`, `maximum`, and closed `enum`. Boolean query params serialize as unquoted `true` or `false`.

### Bytes and blob

- `bytes` supports raw-byte `minLength` and `maxLength`. JSON data represents bytes as `{"$bytes":"<base64>"}`.
- `blob` supports MIME `accept` values and `maxSize`. A MIME wildcard may be a full type suffix such as `image/*`, or `*/*`; partial globs and bare `*` are invalid.

### Array

Requires `items`; optional `minLength` and `maxLength` constrain element count. A union item type means callers cannot assume all elements have the same variant.

### CID link

`cid-link` has no type-specific fields and is encoded as a data-model link. For most application-level and versioned record references, the style guide recommends CIDs in string format rather than binary links.

## Refs, unions, tokens, and unknown

### Ref

A `ref` has one `ref` string. It may point to a local or global named definition, but not to a token, ref, or union. Data for a ref to an object does not include `$type`; data for a referenced record does.

### Union

A `union` has `refs` and optional `closed` (default `false`). Union targets must be `object` or `record` definitions. They cannot target endpoints, subscriptions, unknown, blobs, links, arrays, params, tokens, refs, or other unions.

Union variants normally include `$type`. An open union may have zero refs. A closed union must have at least one ref.

### Token

A `token` is an empty named definition whose description states its meaning. It has no data representation and cannot be targeted by `ref` or `union`. Use fully qualified token references, not local fragments, in string `knownValues` or `enum`.

### Unknown

`unknown` allows any atproto data-model object at that location, but not a primitive or a top-level compound link/blob shape. It cannot be a named def. Avoid `unknown` in record objects unless there is a compelling, explicitly understood reason.

## `$type` rules

Encoded data requires `$type` when content type would otherwise be ambiguous:

- all records include `$type`
- all union variants include `$type`, except top-level subscription messages
- blobs include their protocol `$type`

Use the NSID alone for a main definition, and `nsid#fragment` for a non-main object variant.

## Validation and extra fields

PDS record writes support:

- explicit validation (`validate: true` in TypeScript)
- explicit no validation (`validate: false`)
- optimistic/fail-open validation (`validate` omitted), where known schemas validate and unknown unresolved schemas may be accepted

A successful PDS write reports whether validation occurred. Repository acceptance is therefore not proof that an unknown Lexicon is valid.

Unexpected data fields are ignored or, at worst, warnings under Lexicon v1. Consumers should preserve unknown fields when round-tripping data so older clients do not clobber fields introduced by newer schema versions.
