# .mx Format Specification — Version 1.2

**Status:** Draft
**Version:** 1.2
**Date:** 2026-05-22
**Author:** gist.mx
**MIME Type:** `text/vnd.gist.mx`
**File Extension:** `.mx`

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.2 | 2026-05-22 | Removed `signature` from required fields and core spec. Added `mx_hash` as required field for offline tamper detection. Moved cryptographic signing to `x_` extension pattern. Clarified LF line ending normalization. Made `source`, `duration_seconds`, and `word_count` optional. Added section 9 on application extensions. |
| 1.1 | 2026-03-16 | Added JWS signature metadata object. Made `voice_brief` shape required. |
| 1.0 | 2026-03-05 | Initial release. |

---

## 1. Introduction

The Markdown Experience (`.mx`) format is a superset of markdown that bundles AI-generated comprehension layers alongside the original document content. An `.mx` file enables multiple levels of understanding — from a single-sentence summary to a conversational voice brief — without modifying the source content.

### 1.1 Design Goals

1. **Backward compatibility.** Every `.mx` file is valid markdown with YAML frontmatter. Any markdown-aware tool can render it.
2. **Source linkage.** The `source_hash` field links comprehension layers to the exact source content.
3. **Tamper detection.** The `mx_hash` field enables offline verification that comprehension layers have not been modified since generation.
4. **Model transparency.** The generation model is recorded for reproducibility.
5. **Human readability.** All core field values are natural language prose, ISO timestamps, BCP 47 tags, or SHA-256 hash scalars. No executable instructions or binary-encoded data appear in the core spec.
6. **Forward compatibility.** Unknown frontmatter fields are allowed and ignored by consumers.
7. **Open extensibility.** Application-specific metadata uses the `x_` prefix convention and is outside the scope of this specification.

### 1.2 Terminology

- **Source document:** The original markdown content from which the `.mx` file was derived.
- **Comprehension layer:** A human-readable summary or transformation of source content for a specific consumption context.
- **Frontmatter:** The YAML metadata block delimited by `---` lines at the beginning of the file.
- **Body:** The markdown content following the closing frontmatter delimiter.
- **Producer:** An application that generates `.mx` files.
- **Consumer:** An application that reads and renders `.mx` files.

---

## 2. File Structure

An `.mx` file consists of two parts in this order:

```
---
<YAML frontmatter>
---
<markdown body>
```

### 2.1 Frontmatter

The frontmatter is a YAML 1.2 document enclosed by `---` delimiters on their own lines. It MUST use only the JSON-compatible subset of YAML: scalars, sequences, and mappings. YAML tags, anchors, and aliases are NOT permitted.

### 2.2 Body

The body is the original markdown content, unchanged from the source document. Consumers should process it using CommonMark-compatible behavior, optionally with GitHub Flavored Markdown (GFM) extensions.

### 2.3 Encoding and Line Endings

`.mx` files MUST be encoded as UTF-8 without a byte order mark (BOM).

Line endings MUST be LF (`\n`, 0x0A). Producers MUST emit LF line endings. Consumers receiving `.mx` files via protocols that canonicalize to CRLF (e.g., SMTP) MUST normalize CRLF to LF before processing. This follows the same convention as `text/markdown` per RFC 7763.

---

## 3. Frontmatter Fields

### 3.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `mx_version` | String | Must be exactly `"1.2"`. |
| `generated` | String | RFC 3339 / ISO 8601 date-time of when the file was generated. |
| `source_hash` | String | SHA-256 hash of the source markdown body (excluding any pre-existing frontmatter). Format: `sha256:<64 lowercase hex characters>` |
| `mx_hash` | String | SHA-256 hash of the comprehension layer content. Used for offline tamper detection. See section 4.2. Format: `sha256:<64 lowercase hex characters>` |
| `model` | String | AI model identifier used for generation (e.g., `"qwen3-8b-instruct"`). |
| `oneliner` | String | Single-sentence summary. Maximum 280 characters. |
| `gist` | String | 3–5 sentence summary. No jargon. Maximum 1,000 characters. |
| `eli5` | String | Plain everyday vocabulary explanation. 100–200 words. Analogy-first. Maximum 1,500 characters. |

### 3.2 Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | String | — | Source filename, path, or URL. |
| `language` | String | `"en"` | BCP 47 language tag of the source content. |
| `voice_brief` | Object | — | Voice brief layer. See section 3.3. |
| `manually_edited` | Boolean | `false` | `true` if any comprehension layer has been modified by a human after generation. |
| `edit_history` | Array | `[]` | Ordered list of edit events. Items are open objects. See section 3.4. |
| `guidance` | Object | — | Optional generation guidance metadata. Open object. See section 3.5. |

### 3.3 Voice Brief Object

The `voice_brief` object contains a conversational script written specifically for spoken delivery — not a summary read aloud, but a piece written for the ear with natural cadence and narrative arc.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `script` | String | Yes | Voice-ready script text. 250–350 words recommended. Maximum 3,000 characters. |
| `duration_seconds` | Integer | No | Estimated audio duration in seconds. Minimum: 1. |
| `word_count` | Integer | No | Word count of the script. Minimum: 1. |
| `audio_hash` | String | No | SHA-256 hash of the associated audio file, if generated. Format: `sha256:<64 lowercase hex characters>`. Content hash only — not a URL. |

No additional fields are allowed inside `voice_brief`.

### 3.4 Edit History Array

Each element in `edit_history` records a single edit event. Items are open objects. Recommended fields:

| Field | Type | Description |
|-------|------|-------------|
| `layer` | String | Which layer was edited: `oneliner`, `gist`, `voice_brief`, or `eli5`. |
| `action` | String | `"manual_edit"` or `"regenerate"`. |
| `instruction` | String | User instruction provided when regenerating a layer. |
| `timestamp` | String | ISO 8601 timestamp of the edit. |
| `previous_hash` | String | SHA-256 hash of the layer content before the edit. |

### 3.5 Guidance Object

Records human-provided authoring guidance that shaped the comprehension layers. Open object — all fields are optional and additional fields are allowed.

| Field | Type | Description |
|-------|------|-------------|
| `author` | String | Name of the person who provided guidance. |
| `audience` | String | Intended audience (e.g., `"non-technical executives"`). |
| `key_takeaway` | String | The single most important thing the reader should understand. |
| `tone` | String | Desired tone (e.g., `"technical"`, `"casual"`, `"persuasive"`). |
| `emphasis` | Array of Strings | Topics or sections to emphasize. |
| `skip` | Array of Strings | Topics or sections to de-emphasize or omit. |
| `highlights` | Array of Objects | Section-specific notes. Each has `section` (String) and `note` (String). |
| `lens_template` | String | Identifier of a reusable guidance profile. |

---

## 4. Hash Computation

### 4.1 Source Hash

The `source_hash` is computed as follows:

1. Extract the body content — everything after the closing `---` frontmatter delimiter.
2. Strip any leading or trailing whitespace from the body.
3. Normalize line endings to LF (`\n`).
4. Compute the SHA-256 hash of the resulting UTF-8 byte sequence.
5. Format as `sha256:` followed by 64 lowercase hexadecimal characters.

If the source document itself contains YAML frontmatter, that frontmatter is excluded. Only the markdown body is hashed.

### 4.2 MX Hash

The `mx_hash` enables offline tamper detection without a hosted verification service. It covers both the comprehension layer text and the generation metadata, so neither can be altered independently without invalidating the hash.

Compute `mx_hash` as follows:

1. Concatenate the following field values in this exact order, each separated by a single LF (`\n`):
   - `oneliner`
   - `gist`
   - `voice_brief.script` (use empty string `""` if `voice_brief` is absent)
   - `eli5`
   - `model`
   - `generated`
2. Normalize all internal line endings in each value to LF before concatenation.
3. Compute the SHA-256 hash of the resulting UTF-8 byte sequence.
4. Format as `sha256:` followed by 64 lowercase hexadecimal characters.

A consumer recomputes this hash from the frontmatter values and compares it to the stored `mx_hash`. A mismatch means the file has been modified since generation.

When `manually_edited` is `true`, `mx_hash` reflects the state at the time `manually_edited` was set. Consumers SHOULD surface this distinction to the user.

---

## 5. Multi-File Project MX

When an `.mx` file represents a synthesis of multiple related markdown files, the frontmatter includes:

| Field | Type | Description |
|-------|------|-------------|
| `type` | String | Must be `"project"`. |
| `files` | Array | List of file objects. Each has `path` (String), `oneliner` (String), and `hash` (String: SHA-256 of that file's body). |
| `relationships` | Array | Cross-file relationships. Each has `from` (String), `to` (String), and `type` (String: `"references"`, `"extends"`, or `"contradicts"`). |

The comprehension layers describe the entire project, synthesized across all files.

---

## 6. MIME Type and File Extension

- **MIME type:** `text/vnd.gist.mx`
- **File extension:** `.mx`
- **Encoding:** UTF-8
- **Macintosh File Type Code:** `TEXT`
- **Uniform Type Identifier (macOS):** `com.gistmx.mx-document`
- **Conforms to (macOS UTI):** `public.plain-text`, `net.daringfireball.markdown`

---

## 7. Conformance

### 7.1 MX Producers

A conforming v1.2 producer:

- Emits valid YAML 1.2 frontmatter using only the JSON-compatible subset.
- Includes all required fields from section 3.1.
- Computes `source_hash` per section 4.1.
- Computes `mx_hash` per section 4.2.
- Emits LF line endings throughout the file.
- Preserves the source markdown body without modification.
- Sets `mx_version` to `"1.2"`.

### 7.2 MX Consumers

A conforming consumer:

- Uses a safe YAML loader that does not execute code during parsing.
- Ignores unknown top-level frontmatter fields without error.
- Renders the markdown body using CommonMark-compatible processing.
- Optionally displays comprehension layers from the frontmatter.
- Optionally verifies `source_hash` against the body content.
- Optionally verifies `mx_hash` against the comprehension layer values.
- Normalizes CRLF to LF if received via a CRLF-canonicalizing protocol.

### 7.3 Version Compatibility

v1.0, v1.1, and v1.2 files can coexist in the same ecosystem. Consumers should branch behavior on `mx_version` and remain tolerant of unknown fields. A consumer that does not recognize `.mx` or `text/vnd.gist.mx` SHOULD fall back to `text/markdown` — the file remains valid markdown.

---

## 8. Security Considerations

1. **YAML safety.** Consumers MUST use safe YAML loading. The format does not use YAML tags, anchors, or aliases, but consumers should enforce this to prevent deserialization attacks.
2. **Tamper detection.** `mx_hash` enables consumers to detect post-generation modifications. A mismatch should be surfaced as a warning, not treated as a parse error.
3. **HTML in the body.** The body may contain raw HTML if present in the source. Consumers rendering HTML MUST sanitize it — at minimum stripping `<script>` elements and inline event handler attributes.
4. **Audio hash.** `voice_brief.audio_hash` is a content hash, not a URL. It does not cause automatic network requests. Consumers fetching audio from an external source SHOULD verify the retrieved content against this hash.
5. **Privacy.** Comprehension layers may surface sensitive information from the source in a more accessible form. `guidance.author`, if present, contains personal data subject to applicable data protection regulations.

---

## 9. Application Extensions

The core `.mx` spec contains only natural language prose fields, ISO timestamps, BCP 47 tags, and SHA-256 hash scalars. Application-specific metadata — cryptographic signatures, access control, processing instructions, service identifiers — MUST NOT be added to the core field set.

Applications needing additional metadata SHOULD use the `x_` prefix:

```yaml
x_myapp_signature: "eyJhbGciOiJFZERTQSJ9..."
x_myapp_verified_by: "https://myapp.example/verify/abc123"
```

Extension fields with the `x_` prefix:

- Are out of scope for this specification.
- MUST be ignored by conforming consumers that do not understand them.
- Do not affect `mx_hash` computation.
- Do not affect conformance.

For example, gist.mx may publish a JWS signature as `x_gistmx_sig` to prove a specific file was generated by its service. This is an application-layer concern, not part of the open format definition.

---

## 10. Example

See the [examples](../examples/) directory for complete sample `.mx` files.

---

## 11. Versioning

- **Minor versions** (e.g., 1.2 → 1.3) add optional fields or clarify behavior. All 1.x consumers can read all 1.x files.
- **Major versions** (e.g., 1.x → 2.0) may introduce breaking changes.

Consumers encountering an unknown major version SHOULD warn the user but SHOULD still render the markdown body.
