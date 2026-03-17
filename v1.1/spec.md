# .mx Format Specification — Version 1.1

**Status:** Draft  
**Version:** 1.1  
**Date:** 2026-03-16  
**Author:** gist.mx  
**MIME Type:** `text/vnd.gist.mx`  
**File Extension:** `.mx`

---

## 1. Introduction

The Markdown Experience (`.mx`) format is a superset of markdown that bundles AI-generated comprehension layers alongside the original document content. Version 1.1 adds standardized JWS signature metadata so producers can publish cryptographic authenticity claims for generated frontmatter.

### 1.1 Design Goals

1. **Backward compatibility.** Every `.mx` file is valid markdown with YAML frontmatter.
2. **Source linkage.** The `source_hash` field links comprehension layers to source content.
3. **Model transparency.** The generation model is recorded.
4. **Authenticity metadata.** A structured signature envelope carries JWS metadata for verification workflows.
5. **Forward compatibility.** Unknown frontmatter fields are allowed and ignored by consumers that do not use them.

### 1.2 Terminology

- **Source document:** The original markdown content from which the `.mx` file was derived.
- **Comprehension layer:** A structured summary or transformation of source content.
- **Frontmatter:** The YAML metadata block delimited by `---` lines at the beginning of the file.
- **Body:** The markdown content following the frontmatter.

---

## 2. File Structure

An `.mx` file consists of two parts:

```
---
<YAML frontmatter>
---
<markdown body>
```

### 2.1 Frontmatter

The frontmatter is a YAML 1.2 document enclosed by `---` delimiters on their own lines.

### 2.2 Body

The body is markdown content, typically the source document content. Consumers should process it using CommonMark-compatible markdown behavior, optionally with GitHub Flavored Markdown (GFM) extensions.

### 2.3 Encoding

`.mx` files are UTF-8 encoded. Consumers should accept LF (`\n`) and CRLF (`\r\n`) line endings.

---

## 3. Frontmatter Fields

### 3.1 Required Fields

| Field | Type | Constraint / Description |
|-------|------|--------------------------|
| `mx_version` | String | Must be exactly `"1.1"`. |
| `generated` | String | RFC 3339 / ISO 8601 date-time. |
| `source` | String | Source filename, path, URL, or source identifier. |
| `source_hash` | String | Source content hash string. |
| `model` | String | AI model identifier used for generation. |
| `oneliner` | String | Single-line summary text. |
| `gist` | String | Core summary text. |
| `voice_brief` | Object | Voice layer object. See section 3.3. |
| `eli5` | String | Simplified explanation text. |
| `signature` | Object | Signature metadata object. See section 3.4. |

### 3.2 Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `language` | String | `"en"` | Language code for source/comprehension text. |
| `manually_edited` | Boolean | `false` | Whether a human edited generated layers. |
| `edit_history` | Array | `[]` | Edit metadata entries. Items are open objects. |
| `guidance` | Object | — | Optional generation guidance metadata. Open object. |

### 3.3 Voice Brief Object

The `voice_brief` object is required and must include:

| Field | Type | Required | Constraint / Description |
|-------|------|----------|--------------------------|
| `script` | String | Yes | Voice-ready script text. |
| `duration_seconds` | Integer | Yes | Minimum value: `1`. |
| `word_count` | Integer | Yes | Minimum value: `1`. |
| `audio_hash` | String | No | Optional audio content hash string. |

No additional fields are allowed inside `voice_brief`.

### 3.4 Signature Object

The `signature` object is required and must include:

| Field | Type | Required | Constraint / Description |
|-------|------|----------|--------------------------|
| `type` | String | Yes | Must be exactly `"jws"`. |
| `alg` | String | Yes | Must be exactly `"EdDSA"`. |
| `kid` | String | Yes | Key identifier. Minimum length: `1`. |
| `issuer` | String | Yes | URI identifying signature issuer. |
| `issued_at` | String | Yes | RFC 3339 / ISO 8601 date-time. |
| `claims_version` | String | Yes | Must be exactly `"mxsig-1"`. |
| `jws` | String | Yes | Compact JWS serialization string. Minimum length: `1`. |

No additional fields are allowed inside `signature`.

### 3.5 Extensibility Rules

- Top-level frontmatter allows unknown fields (`additionalProperties: true`).
- `edit_history` items are open objects (`additionalProperties: true`).
- `guidance` is an open object (`additionalProperties: true`).

---

## 4. Source Hash Guidance

The schema requires `source_hash` as a string but does not fix a hash algorithm or pattern. Producers should use stable, documented hashing for interoperability. If using SHA-256, the common representation is:

```
sha256:<64 lowercase hex chars>
```

Consumers should treat `source_hash` as opaque unless they implement a documented verification profile.

---

## 5. MIME Type and File Extension

- **MIME type:** `text/vnd.gist.mx`
- **File extension:** `.mx`

---

## 6. Conformance

### 6.1 MX Producers

A conforming MX producer for version 1.1:

- Emits valid YAML frontmatter.
- Includes all required fields from section 3.1.
- Sets `mx_version` to `"1.1"`.
- Ensures `voice_brief` and `signature` satisfy their object constraints.
- Preserves top-level extensibility for future fields.

### 6.2 MX Consumers

A conforming MX consumer:

- Safely parses YAML frontmatter.
- Ignores unknown top-level fields.
- Handles `voice_brief` and `signature` if present and valid.
- Renders markdown body as markdown.
- May optionally verify `source_hash` and the JWS signature.

### 6.3 Compatibility

Version 1.0 and 1.1 can coexist in the same ecosystem. Consumers should branch behavior by `mx_version` and remain tolerant of unknown fields.

---

## 7. Security Considerations

1. **YAML safety.** Use safe YAML parsing to avoid code execution via deserialization.
2. **Signature verification.** Treat unverified `signature` metadata as informational until cryptographically verified against trusted keys.
3. **Issuer trust.** `issuer` identifies origin but does not imply trust without policy and key validation.
4. **Rendered HTML safety.** Sanitize raw HTML in markdown body when rendering to untrusted environments.

---

## 8. Example

See the [examples](../examples/) directory for complete `.mx` samples, including versioned examples.

---

## 9. Versioning

The `.mx` specification uses semantic versioning. Version 1.1 introduces required signature metadata and required `voice_brief` shape while keeping forward extensibility at the top level.
