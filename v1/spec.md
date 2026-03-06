# .mx Format Specification — Version 1.0

**Status:** Draft
**Version:** 1.0
**Date:** 2026-03-05
**Author:** gist.mx
**MIME Type:** `text/vnd.gist.mx`
**File Extension:** `.mx`

---

## 1. Introduction

The Markdown Experience (`.mx`) format is a superset of markdown that bundles AI-generated comprehension layers alongside the original document content. An `.mx` file enables multiple levels of understanding — from a single-sentence summary to a conversational voice brief — without modifying the source content.

### 1.1 Design Goals

1. **Backward compatibility.** Every `.mx` file is a valid markdown file with YAML frontmatter. Any markdown-aware tool can render it.
2. **Source integrity.** A cryptographic hash links the comprehension layers to the exact source content.
3. **Model transparency.** The AI model used for generation is recorded, enabling reproducibility and comparison.
4. **Human editability.** All fields are plain text. Humans can edit generated layers, and an audit trail records changes.
5. **Forward compatibility.** Unknown frontmatter fields are ignored by parsers. New fields can be added in future versions without breaking existing consumers.

### 1.2 Terminology

- **Source document:** The original markdown content from which the `.mx` file was derived.
- **Comprehension layer:** A structured summary or transformation of the source document for a specific consumption context.
- **Frontmatter:** The YAML metadata block delimited by `---` lines at the beginning of the file.
- **Body:** The markdown content following the frontmatter, identical to the source document.

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

The frontmatter is a YAML 1.2 document enclosed between two `---` delimiters on their own lines. It uses only the JSON-compatible subset of YAML: scalars, sequences, and mappings. No YAML tags, anchors, or aliases are permitted.

### 2.2 Body

The body is the original markdown content, unchanged from the source document. It follows the CommonMark specification with optional GitHub Flavored Markdown (GFM) extensions.

### 2.3 Encoding

`.mx` files are encoded as UTF-8 without a byte order mark (BOM). Line endings may be LF (`\n`) or CRLF (`\r\n`). Consumers must accept both.

---

## 3. Frontmatter Fields

### 3.1 Required Fields

| Field | Type | Description |
|-------|------|-------------|
| `mx_version` | String | Specification version. Must be `"1.0"` for this version. Pattern: `^\d+\.\d+$` |
| `generated` | String | ISO 8601 timestamp of when the `.mx` file was generated. |
| `source_hash` | String | SHA-256 hash of the source markdown content (body only, excluding any pre-existing frontmatter). Format: `sha256:<64 hex characters>` |
| `model` | String | Identifier of the AI model used to generate the comprehension layers. |
| `oneliner` | String | Single-sentence summary. Maximum 280 characters. |
| `gist` | String | 3–5 sentence summary. Maximum 1,000 characters. |
| `eli5` | String | Simplified explanation using vocabulary accessible to a 5-year-old. 100–200 words. Maximum 1,500 characters. |

### 3.2 Optional Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | String | — | Original filename or URL of the source document. |
| `language` | String | — | BCP 47 language tag of the source content (e.g., `"en"`, `"en-US"`, `"ja"`). |
| `voice_brief` | Object | — | Voice brief layer. See section 3.3. |
| `manually_edited` | Boolean | `false` | `true` if any comprehension layer has been modified by a human after generation. |
| `edit_history` | Array | `[]` | Ordered list of edit events. See section 3.4. |
| `guidance` | Object | — | Authoring guidance used to shape the comprehension layers. See section 3.5. |

### 3.3 Voice Brief Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `script` | String | Yes | 250–350 word conversational script written for spoken delivery. Maximum 3,000 characters. |
| `duration_seconds` | Integer | No | Estimated audio duration in seconds (60–180). |
| `word_count` | Integer | No | Word count of the script. |
| `audio_hash` | String | No | SHA-256 hash of the associated audio file, if generated. Format: `sha256:<64 hex characters>` |

### 3.4 Edit History Array

Each element in `edit_history` is an object with:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `layer` | String | Yes | Layer that was edited: `oneliner`, `gist`, `voice_brief`, or `eli5`. |
| `action` | String | Yes | `"manual_edit"` or `"regenerate"`. |
| `instruction` | String | No | User instruction provided for regeneration. |
| `timestamp` | String | No | ISO 8601 timestamp of the edit. |
| `previous_hash` | String | No | SHA-256 hash of the layer content before the edit. |

### 3.5 Guidance Object (v2 Extension)

The `guidance` field enables human-directed comprehension. All sub-fields are optional.

| Field | Type | Description |
|-------|------|-------------|
| `author` | String | Name of the person who provided guidance. |
| `audience` | String | Intended audience (e.g., `"non-technical executives"`, `"new engineers"`). |
| `key_takeaway` | String | The single most important thing the reader should understand. |
| `tone` | String | Desired tone (e.g., `"technical"`, `"casual"`, `"persuasive"`, `"neutral"`). |
| `emphasis` | Array of Strings | Topics or sections to emphasize. |
| `skip` | Array of Strings | Topics or sections to de-emphasize or omit. |
| `highlights` | Array of Objects | Section-specific notes. Each has `section` (String) and `note` (String). |
| `lens_template` | String | Identifier of a reusable guidance profile. |

---

## 4. Source Hash Computation

The `source_hash` is computed as follows:

1. Extract the body content (everything after the closing `---` frontmatter delimiter).
2. Strip any leading or trailing whitespace from the body.
3. Normalize line endings to LF (`\n`).
4. Compute the SHA-256 hash of the resulting UTF-8 byte sequence.
5. Format as `sha256:` followed by 64 lowercase hexadecimal characters.

If the source document contains YAML frontmatter, the frontmatter is excluded from the hash computation. Only the markdown body is hashed.

---

## 5. Multi-File Project MX

When an `.mx` file represents a project (a collection of related markdown files), the frontmatter includes additional fields:

| Field | Type | Description |
|-------|------|-------------|
| `type` | String | Must be `"project"`. |
| `files` | Array | List of file objects, each with `path` (String), `oneliner` (String), and `hash` (String). |
| `relationships` | Array | List of relationship objects, each with `from` (String), `to` (String), and `type` (String: `"references"`, `"extends"`, `"contradicts"`). |

The comprehension layers (`oneliner`, `gist`, `voice_brief`, `eli5`) in a project `.mx` describe the entire project, not any individual file.

---

## 6. MIME Type and File Extension

- **MIME type:** `text/vnd.gist.mx`
- **File extension:** `.mx`
- **Macintosh File Type Code:** `TEXT`
- **Uniform Type Identifier (macOS):** `com.gistmx.mx-document`
- **Conforms to:** `public.plain-text`, `net.daringfireball.markdown`

---

## 7. Conformance

### 7.1 MX Producers

A conforming MX producer:

- Generates valid YAML 1.2 frontmatter using only the JSON-compatible subset.
- Includes all required fields (section 3.1).
- Computes `source_hash` as defined in section 4.
- Preserves the source markdown body without modification.
- Sets `mx_version` to `"1.0"`.

### 7.2 MX Consumers

A conforming MX consumer:

- Parses the YAML frontmatter using a safe YAML loader (no code execution).
- Ignores unknown frontmatter fields without error.
- Renders the markdown body using standard markdown processing.
- Optionally displays comprehension layers from the frontmatter.
- Optionally verifies `source_hash` against the body content.

### 7.3 Backward Compatibility

An MX consumer that does not recognize the `.mx` format or `text/vnd.gist.mx` MIME type should fall back to processing the file as `text/markdown`. The file remains valid and readable.

---

## 8. Security Considerations

1. **YAML parsing.** Consumers MUST use safe YAML loading to prevent code execution through deserialization attacks. No YAML tags, anchors, or aliases are used in the format.
2. **HTML in markdown body.** The body may contain raw HTML if present in the source. Consumers that render HTML should sanitize it (strip `<script>`, event handlers, etc.).
3. **Privacy.** Comprehension layers may expose sensitive information from the source in a more accessible form. The `guidance.author` field contains personal data subject to applicable data protection regulations.
4. **External references.** The `voice_brief.audio_hash` is a content hash, not a URL. It does not create automatic external references. Consumers resolving audio should verify the hash matches retrieved content.

---

## 9. Example

See the [examples](../examples/) directory for complete sample `.mx` files.

---

## 10. Versioning

This specification follows semantic versioning. Minor versions (e.g., 1.1) add optional fields. Major versions (e.g., 2.0) may introduce breaking changes. The `mx_version` field in each file indicates the specification version.

Consumers should parse `mx_version` and handle unknown major versions gracefully (e.g., display a warning but still render the markdown body).
