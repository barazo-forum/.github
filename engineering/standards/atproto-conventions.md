# AT Protocol Conventions for Barazo

Condensed reference for AT Protocol compliance during lexicon design, API implementation, and code review. Derived from the [AT Protocol specification](https://atproto.com/specs/lexicon) and [data model](https://atproto.com/specs/data-model), verified against canonical Bluesky lexicons.

**When to consult this file:** Before writing or reviewing any lexicon schema, firehose handler, PDS interaction, or record validation code.

**When this file is insufficient:** For edge cases or protocol changes, check the source:
- Spec: `https://atproto.com/specs/lexicon` and `https://atproto.com/specs/data-model`
- Canonical lexicons: `https://github.com/bluesky-social/atproto/tree/main/lexicons`
- Fetch a single raw file from GitHub for comparison (cheaper than Context7)

---

## 1. Lexicon JSON File Structure

```json
{
  "lexicon": 1,
  "id": "forum.barazo.topic.post",
  "description": "Optional short overview.",
  "defs": {
    "main": { ... },
    "namedDef": { ... }
  }
}
```

| Field | Required | Notes |
|-------|----------|-------|
| `lexicon` | Yes | Always `1` (current version) |
| `id` | Yes | NSID matching the file path |
| `description` | No | Short overview of the lexicon |
| `defs` | Yes | Map of definition names to schema objects |

- `main` is the primary definition, referenced by NSID alone
- Other named defs are referenced as `forum.barazo.topic.post#namedDef`
- Each def must have a `type` field

---

## 2. Primary Types

### `record` (what Barazo uses for PDS-stored data)

```json
{
  "type": "record",
  "description": "Record containing a ...",
  "key": "tid",
  "record": {
    "type": "object",
    "required": ["field1", "field2"],
    "properties": { ... }
  }
}
```

- `key` (string, required): Record key type -- see section 5
- `record` (object, required): Must be type `object`

### `query` -- XRPC GET endpoint
### `procedure` -- XRPC POST endpoint
### `subscription` -- WebSocket event stream

Barazo uses `record` for lexicons. Query/procedure/subscription are for XRPC endpoints (defined in `com.atproto.*`, not in app lexicons).

---

## 3. Field Types

### Primitives

| Type | Key fields | Notes |
|------|-----------|-------|
| `string` | `format`, `maxLength`, `maxGraphemes`, `minLength`, `minGraphemes`, `enum`, `knownValues`, `default`, `const` | `maxLength` = UTF-8 bytes. `maxGraphemes` = Unicode Grapheme Clusters. |
| `integer` | `minimum`, `maximum`, `enum`, `default`, `const` | Signed 64-bit, but limit to 53 bits for JS safety. |
| `boolean` | `default`, `const` | |
| `bytes` | `minLength`, `maxLength` | Raw byte count. JSON: `{"$bytes": "<base64>"}` |

### Complex

| Type | Key fields | Notes |
|------|-----------|-------|
| `object` | `properties`, `required`, `nullable` | Generic nested object. |
| `array` | `items`, `minLength`, `maxLength` | `maxLength` = max element count. `items` = element schema. |
| `blob` | `accept`, `maxSize` | `accept`: MIME types with `*` globs (e.g., `image/*`). `maxSize`: bytes. |
| `cid-link` | (none) | Content identifier reference. JSON: `{"$link": "<cid>"}` |

### References

| Type | Key fields | Notes |
|------|-----------|-------|
| `ref` | `ref` (string, required) | Points to one specific definition. Cannot ref `token`, `ref`, or `union`. |
| `union` | `refs` (array, required), `closed` (bool, optional) | Multiple possible types. Default `closed: false` (extensible). |
| `token` | (none) | Named empty value. Can only be referenced in `knownValues`/`enum` strings. |
| `unknown` | (none) | Any valid data model value. Use sparingly. |

---

## 4. String Formats

Use `"format"` on string fields for semantic validation:

| Format | Validates as | Example |
|--------|-------------|---------|
| `did` | Generic DID | `did:plc:abc123` |
| `handle` | AT Protocol handle | `user.bsky.social` |
| `at-uri` | AT URI | `at://did:plc:abc/app.bsky.feed.post/rkey` |
| `at-identifier` | Handle OR DID | Either of the above |
| `nsid` | Namespaced identifier | `app.bsky.feed.post` |
| `cid` | Content identifier (string) | `bafyrei...` |
| `tid` | Timestamp identifier | `3jzfcijpj2z2a` |
| `record-key` | General record key syntax | See section 5 |
| `datetime` | ISO 8601 with timezone | `2026-02-12T10:00:00.000Z` |
| `uri` | Generic URI (RFC-3986) | `https://example.com` |
| `language` | BCP 47 language tag | `en`, `nl`, `en-US` |

---

## 5. Record Key Types

Specified in the lexicon `"key"` field:

| Key type | When to use | Example |
|----------|------------|---------|
| `tid` | Most records. Timestamp-based, unique per collection. | `3jzfcijpj2z2a` |
| `literal:self` | Singleton records (one per user). | Profile, preferences |
| `nsid` | When key must be a valid NSID. | Rare |
| `any` | Flexible. Domain names, integers, AT URIs. | Specialized cases |

**Record key constraints (all types):**
- Characters: `A-Za-z0-9` `.` `-` `_` `:` `~`
- Length: 1-512 characters
- Prohibited: `.` and `..` alone
- Case-sensitive
- Must be valid URI path components

---

## 6. The strongRef Pattern

All record-to-record references use `com.atproto.repo.strongRef`:

```json
{
  "type": "ref",
  "ref": "com.atproto.repo.strongRef",
  "description": "The thing being referenced."
}
```

The strongRef object contains `{ uri, cid }` -- an AT URI plus a content hash. This is universal across the ecosystem (Bluesky, Frontpage, WhiteWind, all community apps).

**When to use:** Any field that points to another record (parent, root, subject, target).

**When NOT to use:** DID references (just use `"format": "did"` on a string), AT URI without integrity check (use `"format": "at-uri"` on a string).

---

## 7. Self-Labels Pattern

Content warnings / maturity labels follow Bluesky's pattern:

```json
"labels": {
  "type": "union",
  "description": "Self-label values for content maturity.",
  "refs": ["com.atproto.label.defs#selfLabels"]
}
```

- Use `union` (not `ref`) for forward compatibility
- The `selfLabels` type contains `{ values: [{ val: string }] }`, max 10 entries
- Standard label values: `sexual`, `nudity`, `graphic-media`, `porn`

---

## 8. String Length Conventions

Two independent constraints work together:

| Constraint | Measures | Purpose |
|-----------|---------|---------|
| `maxLength` | UTF-8 bytes | Prevents storage overflow. Required for all strings with limits. |
| `maxGraphemes` | Unicode Grapheme Clusters | Prevents UI overflow. Loosely = "visible characters". |

**Ecosystem ratio:** ~10:1 (bytes to graphemes). Examples from Bluesky:
- Display name: `maxGraphemes: 64`, `maxLength: 640`
- Post text: `maxGraphemes: 300`, `maxLength: 3000`
- Tags: `maxGraphemes: 64`, `maxLength: 640`

**Barazo convention:**
- Short text (titles, tags, names): specify both `maxGraphemes` and `maxLength`
- Long-form content (post body, reply body): `maxLength` only (following WhiteWind)
- `minLength` is always in UTF-8 bytes

---

## 9. Timestamps

All `createdAt` fields are **client-declared**:
- The description should say: `"Client-declared timestamp when this [thing] was originally created."`
- AppViews must NOT trust these for ordering -- use server-side `indexedAt` instead
- Format: `"format": "datetime"` (ISO 8601 with timezone)

---

## 10. Unknown Fields and Validation

**Lenient by default:** Unexpected fields in records should be ignored (treated as warnings at worst). Third parties can add fields to records.

**Three validation modes:**
1. **Explicit validation** (`validate: true`): record fails if Lexicon unknown
2. **No validation** (`validate: false`): records allowed without checks
3. **Optimistic** (default): validates if Lexicon known locally, allows if unknown

**Barazo policy:** Always validate with explicit mode for our own lexicons. Use Zod schemas from `@singi-labs/barazo-lexicons` at both API endpoints and firehose ingestion.

---

## 11. Data Model Rules

- **No floats.** Integers only (signed 64-bit, 53-bit for JS safety).
- **`$`-prefixed fields are reserved:** `$type`, `$bytes`, `$link`. Never define custom `$` fields.
- **Null vs missing:** Semantically different. Omitting a field != setting it to `null` != setting it to a falsy value.
- **Object keys** are always strings.
- **CID format:** Version 1, SHA-256, base32 with `b` prefix.

---

## 12. Validation Tooling (CI Integration)

### `@atproto/lex-cli` (primary)
- Validates lexicon JSON against the AT Protocol meta-schema
- Generates TypeScript types from lexicons
- Installed as dev dependency in barazo-lexicons

```bash
# Validate schemas
./node_modules/.bin/lex validate ./lexicons/**/*.json

# Generate TypeScript
./node_modules/.bin/lex gen-server ./src/lexicon ./lexicons/*
```

### `goat` CLI (supplementary linting)
- Lints lexicon files for best practices (e.g., missing maxLength)
- Can pull and compare against published lexicons

```bash
goat lex lint lexicons/forum/barazo/topic/post.json
goat lex pull app.bsky.feed.post  # fetch canonical for comparison
```

### Bluesky interop tests
- Repo: `github.com/bluesky-social/atproto-interop-tests`
- Reusable test files for spec compliance
- Reference for edge cases in TID generation, CID computation, handle resolution

---

## 13. Common Mistakes

| Mistake | Correct approach |
|---------|-----------------|
| Inline `{ uri, cid }` instead of strongRef | Use `"ref": "com.atproto.repo.strongRef"` |
| `"type": "ref"` for labels | Use `"type": "union", "refs": [...]` |
| `maxLength` only (no `maxGraphemes`) on short text | Add both for Unicode safety |
| `"key": "tid"` on singleton records | Use `"key": "literal:self"` |
| Trusting `createdAt` for ordering | Use server-side `indexedAt` |
| `$type` in lexicon properties | Omitted -- added automatically by protocol |
| Defining custom `$`-prefixed fields | Reserved for protocol use |
| Floats in record schemas | Use `integer` only |
| Required field that should be optional | Adding required fields later is a breaking change |
| `contentFormat: "markdown"` as required | Make optional with default if only one value exists |

---

## 14. Quick Reference: Barazo Lexicons

| Record type | Key | Required fields |
|------------|-----|----------------|
| `forum.barazo.topic.post` | `tid` | title, content, community, category, createdAt |
| `forum.barazo.topic.reply` | `tid` | content, root, parent, community, createdAt |
| `forum.barazo.interaction.reaction` | `tid` | subject, type, community, createdAt |
| `forum.barazo.interaction.vote` | `tid` | subject, direction, community, createdAt *(schema defined, not yet implemented in AppView)* |
| `forum.barazo.actor.preferences` | `literal:self` | maturityLevel, updatedAt |

**Canonical schemas:** `specs/prd-lexicons.md` section 4.

---

**Last verified against:** AT Protocol spec (atproto.com, February 2026)
**Canonical spec URL:** https://atproto.com/specs/lexicon
