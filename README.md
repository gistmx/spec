# .mx Format Specification

The **Markdown Experience** (`.mx`) format extends standard markdown with structured AI-generated comprehension layers embedded in YAML frontmatter. It makes markdown documents accessible by providing multiple levels of understanding — from a single-sentence summary to a conversational voice brief — without breaking backward compatibility.

## Quick Overview

An `.mx` file is a valid markdown file. Any tool that reads `.md` can read `.mx`. The comprehension layers live in YAML frontmatter and are invisible to unaware renderers.

```yaml
---
mx_version: "1.0"
generated: "2026-03-05T12:00:00Z"
source: "README.md"
source_hash: "sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
model: "qwen3-8b-instruct"

oneliner: >
  A single sentence that captures the entire document.

gist: >
  Three to five sentences. The elevator pitch. What you'd say if someone
  asked "what's this about?" No jargon.

voice_brief:
  script: >
    A 250-350 word conversational script written specifically for spoken
    delivery. Not a summary read aloud — a purpose-written voice piece
    with narrative arc and natural cadence.
  duration_seconds: 120
  word_count: 290

eli5: >
  The whole thing explained like you're talking to a curious five-year-old.
  Simple words, fun analogies, ends with a question.
---

# Original Markdown Content

Everything below the frontmatter is the original document, unchanged.
```

## Five Comprehension Layers

| Layer | Name | What It Is |
|-------|------|-----------|
| 1 | **One-Liner** | Single sentence, max 280 characters. The tweet-length version. |
| 2 | **Core Gist** | 3–5 sentences. Elevator pitch. No jargon. |
| 3 | **Voice Brief** | 250–350 word conversational script, purpose-written for audio delivery. |
| 4 | **ELI5** | 100–200 words. Kid-friendly. Analogy-first. Wonder-preserving. |
| 5 | **Full Render** | The original markdown, beautifully rendered. |

## Key Properties

- **Backward compatible.** Valid markdown. Opens in any editor.
- **Source-linked.** SHA-256 hash verifies the layers match the content.
- **Model-stamped.** The AI model is recorded for reproducibility.
- **Human-editable.** A `manually_edited` flag indicates human refinement.

## Repository Structure

```
├── README.md              ← You are here
├── LICENSE                ← CC BY 4.0
├── v1/
│   ├── spec.md            ← Format specification (v1.0)
│   └── schema.json        ← JSON Schema for frontmatter validation
├── examples/
│   └── vercel-react-best-practices.mx
├── website/               ← Static site for gist.mx (deploy to Vercel)
│   ├── index.html
│   └── vercel.json
└── docs/
    ├── gist_mx_technical_spec_v3.md
    └── iana_media_type_registration.md
```

## Specification

- [Format Specification (v1.0)](./v1/spec.md) — Full format definition
- [JSON Schema](./v1/schema.json) — Machine-readable frontmatter schema
- [Examples](./examples/) — Sample `.mx` files

## MIME Type

```
text/vnd.gist.mx
```

Registered with [IANA](https://www.iana.org/assignments/media-types/) under the vendor tree.

File extension: `.mx`

## Tools

- [gist.mx](https://gist.mx) — Web service for generating `.mx` files
- `@gistmx/cli` — Command-line tool (`npm install -g @gistmx/cli`)
- `gistmx/convert-action` — GitHub Action for auto-generating `.mx` on push

## License

This specification is released under [CC BY 4.0](./LICENSE). You are free to implement `.mx` parsers, generators, and viewers without restriction.
