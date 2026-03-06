# gist.mx Technical Specification v3

## 0. Product Vision and Competitive Positioning

### 0.1 What gist.mx Is

gist.mx is a **markdown comprehension platform** that transforms `.md` files into enriched `.mx` files — a durable, shareable format that bundles AI-generated understanding alongside the original content. It produces layered comprehension (one-liner, gist, voice brief, ELI5), not a single summary. It speaks your document aloud. It synthesizes entire projects, not just individual files. And over time, it becomes an authoring tool where humans guide the AI's understanding to produce briefings that reflect their expertise and point of view.

### 0.2 What gist.mx Is NOT

gist.mx is not a general-purpose summarization tool. It does not summarize web pages, YouTube videos, podcasts, or arbitrary URLs. It does not compete with tools like `steipete/summarize`, which already handle the "point at anything and get the gist" use case with 3.7K stars, a Chrome sidebar, CLI distribution via npm and Homebrew, multi-model support, and media transcription pipelines.

### 0.3 Competitive Landscape

| Capability | steipete/summarize | TLDR This / QuillBot | gist.mx |
|-----------|-------------------|---------------------|---------|
| Summarize any URL | ✅ (core feature) | ✅ | ❌ (not in scope) |
| YouTube / podcast / audio / video | ✅ (Whisper, yt-dlp, OCR) | ❌ | ❌ (not in scope) |
| Chrome sidebar extension | ✅ | ✅ (Chrome extension) | ❌ (not in scope) |
| CLI tool (npm, Homebrew) | ✅ | ❌ | ✅ (markdown-specific) |
| Multi-model support | ✅ (OpenAI, Anthropic, Google, xAI, local, OpenRouter) | ❌ (proprietary) | ✅ (open-source self-hosted + API fallback) |
| **Durable output file format** | ❌ (ephemeral output) | ❌ (ephemeral output) | **✅ (.mx format)** |
| **Multi-layer comprehension** | ❌ (single summary at configurable length) | ❌ (single summary) | **✅ (5 layers: one-liner, gist, voice, ELI5, full doc)** |
| **Voice generation (TTS)** | ❌ | ❌ | **✅ (Kokoro, purpose-written voice script)** |
| **Multi-file project synthesis** | ❌ (one input at a time) | ❌ | **✅ (cross-file analysis, relationship maps, meta-summary)** |
| **Hosted web viewer** | ❌ (no hosted UI) | ❌ | **✅ (gist.mx/v/{hash}, shareable links)** |
| **Mobile-native reader app** | ❌ | ❌ | **✅ (iOS + Android)** |
| **Guided authoring (Lens)** | ❌ | ❌ | **✅ (v2 — audience, tone, emphasis, human-edited layers)** |
| **GitHub Action for auto-generation** | ❌ | ❌ | **✅ (commit .mx alongside .md)** |

### 0.4 Differentiation Summary

gist.mx differentiates on four axes that no existing tool addresses:

1. **Format, not function.** Summarize produces ephemeral text. gist.mx produces a `.mx` file — a durable artifact you can commit, share, version, and distribute. The file IS the product.

2. **Comprehension layers, not compression levels.** Existing tools produce one summary at a configurable length. gist.mx produces five layers designed for five consumption contexts: a tweet (one-liner), an elevator pitch (gist), a podcast segment (voice brief), a kid-friendly explainer (ELI5), and the full rendered document. These are different registers of understanding, not different word counts.

3. **Voice as first-class output.** No summarization tool generates audio. gist.mx produces a purpose-written conversational voice script (not TTS-on-summary) and generates audio embedded in the `.mx` file. The voice brief is crafted for the ear, not the eye.

4. **Project intelligence.** Existing tools work on one input. gist.mx understands how multiple markdown files relate to each other, eliminates redundancy across files, infers hierarchy, detects contradictions, and produces a single project-level briefing.

### 0.5 Collaboration Opportunity

Since `steipete/summarize` is built by the Openclaw founder, and Clawware (the skill registry you're building) operates in the Openclaw ecosystem, there's a natural collaboration path. Summarize could be the extraction and general summarization engine. gist.mx could be the format, viewer, and distribution layer specifically for markdown comprehension. These products complement rather than compete.

---

## 1. Product Roadmap: Three Versions

### v1 — Auto-Generated Understanding (Launch)

The AI reads your markdown and produces its best understanding as five comprehension layers. No human guidance required. Users can manually edit the generated `.mx` file in any text editor after the fact.

**Scope:**
- Single-file and multi-file `.md` → `.mx` conversion
- Five comprehension layers (one-liner, gist, voice brief, ELI5, full render)
- Voice audio generation (Kokoro TTS)
- Web viewer with sidebar, audio player, shareable links
- Mobile reader app (iOS + Android)
- CLI tool and GitHub Action
- API for agents and CI pipelines

### v1.5 — Inline Editing (Fast Follow, +2–3 weeks after launch)

Users can edit AI-generated layers directly in the web viewer. Click any layer, modify the text, save. The `.mx` file updates with `manually_edited: true`. This is the minimum viable human-in-the-loop.

**Scope (additive to v1):**
- Inline contenteditable fields for each comprehension layer in the web viewer
- Save/update edited `.mx` to R2
- Regenerate individual layers on demand ("try again" button per layer)
- `manually_edited` and `edit_history` fields populated in frontmatter
- Mobile: edit layers in the reader app

### v2 — Lens: Guided Comprehension Authoring (Major release, +6–8 weeks after v1.5)

The user guides the AI's understanding before and after generation. The output reflects the author's expertise, not just the AI's default compression. This transforms gist.mx from a summarization tool into a **comprehension authoring platform**.

**Scope (additive to v1.5):**
- Pre-generation guidance: audience selector, tone picker, key takeaway prompt, emphasis/skip controls
- Source annotation: highlight important sections, strike through irrelevant sections, add margin notes
- Post-generation refinement: regenerate layers with custom instructions, pick between variants, lock approved layers
- Author attribution: `guidance.author` field in `.mx` frontmatter
- Full guidance and edit history stored in `.mx` for provenance
- Lens templates: save and reuse guidance profiles ("Executive Brief," "Developer Onboarding," "Investor Deck")

---

## 2. System Overview

gist.mx converts markdown files (`.md`) into enriched Markdown Experience files (`.mx`) by generating AI comprehension layers: a one-liner, core gist, 2-minute voice brief (text + audio), and a 5-year-old explainer. The service operates as a web application, a native mobile app (iOS + Android), a public REST API, a CLI tool, and a GitHub Action.

### 2.1 Architecture Pattern

The system follows a **queue-driven microservices architecture** with three core services, a shared object store, and a task queue.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                                 │
├───────────┬──────────┬──────────┬────────────┬─────────────────────┤
│  Web App  │ iOS App  │ Android  │  CLI Tool  │  GitHub Action      │
│  (React)  │ (React   │  App     │  (Node.js) │  (Node.js)          │
│           │  Native) │ (RN)    │            │                     │
└─────┬─────┴────┬─────┴────┬─────┴──────┬─────┴──────────┬──────────┘
      │          │          │            │                │
      ▼          ▼          ▼            ▼                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     API GATEWAY (Cloudflare Workers)                │
│  Rate limiting · Auth (JWT) · Request validation · CORS            │
├─────────────────────────────────────────────────────────────────────┤
│                     TASK QUEUE (BullMQ on Redis)                    │
│  Job scheduling · Priority queues · Retry logic · Dead letter      │
├──────────┬───────────────────┬──────────────────────────────────────┤
│          │                   │                                      │
│          ▼                   ▼                                      │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐          │
│  │  SUMMARIZER   │  │  TTS ENGINE   │  │  MX BUNDLER   │          │
│  │  SERVICE      │  │  SERVICE      │  │  SERVICE      │          │
│  │               │  │               │  │               │          │
│  │  LLM inference│  │  Voice synth  │  │  YAML + MD    │          │
│  │  (text layers)│  │  (audio gen)  │  │  assembly     │          │
│  └───────┬───────┘  └───────┬───────┘  └───────┬───────┘          │
│          │                   │                   │                  │
│          ▼                   ▼                   ▼                  │
│  ┌─────────────────────────────────────────────────────────┐       │
│  │              OBJECT STORE (Cloudflare R2)                │       │
│  │  .mx files · .mp3 audio · cached results · source .md  │       │
│  └─────────────────────────────────────────────────────────┘       │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────┐       │
│  │              METADATA STORE (SQLite / Turso)             │       │
│  │  Jobs · Users · Usage · Rate limits · Analytics          │       │
│  └─────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 Design Principles

1. **Stateless compute.** Every service is stateless; all state lives in R2 (objects), Redis (queue), or Turso (metadata).
2. **Queue-first processing.** All conversion jobs go through BullMQ. This decouples request acceptance from processing, enables retry, and allows horizontal scaling.
3. **Model-agnostic AI layer.** The summarizer service abstracts model selection behind an interface. Swapping models requires a config change, not a code change.
4. **Offline-first mobile.** The mobile apps cache rendered `.mx` files and audio locally. Full functionality for previously converted files without network access.
5. **Cost ceiling.** Infrastructure stays under $50/month at low-to-moderate traffic (up to 5,000 conversions/month). Scales linearly, not exponentially.
6. **v1 output is v2 input.** Every auto-generated `.mx` file (v1) is immediately editable (v1.5) and guidable (v2). No migration, no format change. The schema is forward-compatible from day one.

---

## 3. AI Model Strategy

### 3.1 Summarization — LLM Selection

The summarization task (generating one-liner, gist, voice script, ELI5) is a structured text generation problem. It requires strong instruction following, compression ability, and tone control. It does not require massive context windows for most files (typical markdown files are 1K–20K tokens).

#### Recommended Model Tier List

| Tier | Model | Parameters | Context Window | License | Hardware | Use Case |
|------|-------|-----------|----------------|---------|----------|----------|
| **Primary (self-hosted)** | **Qwen3-8B-Instruct** | 8B (3B active MoE) | 128K tokens | Apache 2.0 | 1× A10G (24GB VRAM) or 1× L4 | Production default. Strong summarization, instruction following, multilingual. Runs on a single mid-range GPU. |
| **Lightweight / Edge** | **SmolLM3-3B** | 3B | 64K tokens | Apache 2.0 | 1× T4 (16GB VRAM) or CPU (slow) | Budget tier, high-volume batch processing, or fallback when primary GPU is saturated. |
| **Ultra-light / CPU-only** | **Qwen3-0.6B** | 0.6B | 32K tokens | Apache 2.0 | CPU only (8GB RAM) | One-liner and gist generation only (not voice script or ELI5). On-device mobile inference future path. |
| **Premium fallback (API)** | **Claude Sonnet 4.5** | — | 200K tokens | API (pay-per-use) | None (cloud API) | Overflow, very large files (>50K tokens), quality-critical regenerations, or Lens guided generation (v2). |
| **Premium fallback (API)** | **DeepSeek-V3.2** | 671B MoE | 128K tokens | MIT | None (cloud API) | Cost-effective API alternative at ~$0.27/M input tokens. |

#### Inference Infrastructure

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| Inference server | **vLLM** | Industry-standard for self-hosted LLM serving. PagedAttention for efficient KV cache, continuous batching for throughput, OpenAI-compatible API. |
| GPU hosting | **RunPod** or **Modal** serverless GPU | Pay-per-second GPU billing. No idle cost. Spin up A10G instances on demand. RunPod starts at ~$0.39/hr for A10G. |
| Model format | **AWQ 4-bit quantized** | Reduces VRAM requirements by ~4x with minimal quality loss. Qwen3-8B-AWQ fits comfortably in 8GB VRAM. |
| Fallback | **Anthropic API / DeepSeek API** | If self-hosted inference is down or queue is backed up >5 minutes, route to API fallback. |

#### Prompt Architecture

**v1 (Auto-generated):** A single inference call generates all four text layers using structured JSON output:

```
System: You are a document comprehension engine. Given a markdown document,
generate four comprehension layers. Respond ONLY with valid JSON.

{
  "oneliner": "<single sentence, max 30 words, captures the entire document>",
  "gist": "<3-5 sentences, elevator pitch, no jargon>",
  "voice_brief": "<250-350 words, conversational, written for spoken delivery,
                   narrative arc: situation → what this does → why it matters → 
                   what's interesting. No parentheticals, no complex nesting, 
                   no lists longer than 3 items.>",
  "eli5": "<100-200 words, vocabulary ceiling of a 5-year-old, analogy-first,
            every abstract concept mapped to something physical/familiar,
            ends with a question or hook>"
}

User: <full markdown document content>
```

**v2 (Lens — guided generation):** The prompt is augmented with guidance constraints:

```
System: You are a document comprehension engine. A human author has provided
guidance about how this document should be understood and communicated.
Follow their guidance precisely. Respond ONLY with valid JSON.

Guidance:
- Audience: {audience}
- Key takeaway: {key_takeaway}
- Tone: {tone}
- Emphasize: {emphasis_list}
- De-emphasize or skip: {skip_list}
- Author notes on specific sections:
  {highlighted_sections_with_notes}

Generate layers that reflect the AUTHOR'S understanding, not a generic summary.
The output should read as if the author wrote it themselves.

{same JSON schema as v1}

User: <full markdown document content>
```

**v1.5 (Per-layer regeneration):** When a user clicks "regenerate" on a specific layer:

```
System: Regenerate ONLY the {layer_name} for this document.
Keep the other layers unchanged. Additional instruction from the user:
"{user_instruction}"

Current layers (for context — do not modify these):
- oneliner: {current_oneliner}
- gist: {current_gist}
- voice_brief: {current_voice_brief}
- eli5: {current_eli5}

Regenerate ONLY {layer_name}. Respond with just the text for that layer.

User: <full markdown document content>
```

Token budget per conversion (Qwen3-8B):
- Input: ~2K–15K tokens (typical markdown file) + ~200 tokens (guidance, v2)
- Output: ~500–800 tokens (all four layers combined)
- Total: ~3K–16K tokens per conversion
- Estimated cost (self-hosted RunPod A10G): ~$0.001–$0.004 per conversion
- Estimated cost (API fallback, Claude Sonnet): ~$0.01–$0.05 per conversion

### 3.2 Text-to-Speech — Model Selection

#### Recommended Model Tier List

| Tier | Model | Parameters | License | Hardware | Quality |
|------|-------|-----------|---------|----------|---------|
| **Primary (self-hosted)** | **Kokoro v1.0** | 82M | Apache 2.0 | CPU only (runs on Raspberry Pi) | High (TTS Arena top-5) |
| **Premium quality** | **Chatterbox** | ~300M | MIT | 1× T4 GPU (16GB VRAM) | Very high (beats ElevenLabs in blind tests) |
| **API fallback** | **ElevenLabs** | — | API (pay-per-use) | None | Premium |
| **Browser-native fallback** | **Web Speech API** | — | Built-in | Client device | Low-medium |

#### TTS Infrastructure

```python
from kokoro import KPipeline

pipeline = KPipeline(lang_code='a')  # 'a' for American English
audio_segments = pipeline(voice_brief_text, voice='af_heart', speed=1.0)
# Concatenate segments → export as MP3 at 24kHz, 128kbps CBR
```

Audio output: MP3 (128kbps), 100–140 seconds duration, ~250KB–500KB per file.

---

## 4. .MX File Format Specification

### 4.1 Overview

An `.mx` file is valid YAML frontmatter + markdown body. Any standard markdown viewer renders it. An MX-aware viewer unlocks the full layered experience.

MIME type: `text/markdown+mx` (with `text/markdown` fallback)
File extension: `.mx`

### 4.2 Frontmatter Schema

The schema is forward-compatible across all three versions. v1 populates the core fields. v1.5 adds `manually_edited` and `edit_history`. v2 adds `guidance`.

```yaml
---
# === Core (v1) ===
mx_version: "1.0"
generated: "2026-03-01T12:00:00Z"
source: "Clawware_v4.md"
source_hash: "sha256:abc123..."
model: "qwen3-8b-instruct"
language: "en"

oneliner: >
  Single sentence, max 280 characters.

gist: >
  3-5 sentences. Elevator pitch.

voice_brief:
  script: >
    250-350 words. Conversational. Written for spoken delivery.
  duration_seconds: 120
  word_count: 290
  audio_hash: "sha256:def456..."

eli5: >
  100-200 words. Kid-friendly. Analogy-first.

# === Editing (v1.5) ===
manually_edited: false                    # true if any layer was human-edited
edit_history:                              # array of edit events
  - layer: "oneliner"
    action: "regenerated"                  # regenerated | manually_edited
    instruction: "Focus on cost savings"   # user instruction (if regenerated)
    timestamp: "2026-03-02T10:00:00Z"
    previous_hash: "sha256:..."            # hash of previous layer content

# === Guidance / Lens (v2) ===
guidance:
  author: "jordan"                         # who created this guided .mx
  audience: "non-technical executives"
  key_takeaway: "This reduces onboarding from weeks to hours at near-zero cost"
  tone: "persuasive but honest"
  emphasis:
    - "business value"
    - "timeline"
    - "cost"
  skip:
    - "CI/CD pipeline details"
    - "XML schema specifics"
  highlights:
    - section: "## 1. Product Scope"
      note: "Lead with this for the board"
    - section: "## 13. Operations"
      note: "They'll ask about cost — emphasize the $0-50/month number"
  lens_template: "executive-brief"         # reusable guidance profile (if used)
---

# Original Markdown Content
[... unchanged source markdown ...]
```

### 4.3 Schema Versioning

| Field Group | Populated in v1 | Populated in v1.5 | Populated in v2 |
|------------|----------------|-------------------|-----------------|
| `mx_version`, `generated`, `source`, `source_hash`, `model`, `language` | ✅ | ✅ | ✅ |
| `oneliner`, `gist`, `voice_brief`, `eli5` | ✅ (auto) | ✅ (auto + edits) | ✅ (guided) |
| `manually_edited`, `edit_history` | ❌ (always false, empty) | ✅ | ✅ |
| `guidance` | ❌ (absent) | ❌ (absent) | ✅ |

Parsers must treat all v1.5 and v2 fields as optional. A v1-generated `.mx` file is valid input for v1.5 editing or v2 Lens authoring — no migration needed.

### 4.4 Multi-File Project MX

When processing a folder/repo, a meta `.mx` is generated with additional frontmatter:

```yaml
mx_version: "1.0"
type: "project"
files:
  - path: "README.md"
    oneliner: "..."
    hash: "sha256:..."
  - path: "docs/architecture.md"
    oneliner: "..."
    hash: "sha256:..."
relationships:
  - from: "README.md"
    to: "docs/architecture.md"
    type: "references"
```

---

## 5. Backend Architecture

### 5.1 API Gateway (Cloudflare Workers)

**Runtime:** Cloudflare Workers (V8 isolates, 0ms cold start)
**Framework:** Hono 4.x

**Key endpoints:**

```
POST   /api/v1/convert              Single file conversion (v1)
POST   /api/v1/convert-batch        Multi-file / folder conversion (v1)
POST   /api/v1/convert-repo         GitHub repo conversion (v1)
GET    /api/v1/jobs/{id}            Job status + results
GET    /api/v1/jobs/{id}/stream     SSE stream for real-time progress
GET    /api/v1/mx/{hash}            Retrieve hosted .mx file
GET    /api/v1/mx/{hash}/audio      Stream/download voice brief audio
GET    /api/v1/mx/{hash}/embed      Embeddable widget (iframe)
PATCH  /api/v1/mx/{hash}/layers     Update individual layers (v1.5)
POST   /api/v1/mx/{hash}/regenerate Regenerate specific layer (v1.5)
POST   /api/v1/convert-guided       Guided conversion with Lens (v2)
POST   /auth/token                  API key → JWT exchange
GET    /health                      Service health check
```

### 5.2 Task Queue (BullMQ + Upstash Redis)

| Queue | Priority | Purpose |
|-------|----------|---------|
| `summarize` | High | LLM inference for text layers |
| `tts` | Medium | Voice synthesis |
| `bundle` | Low | Assemble .mx file + upload to R2 |
| `repo-crawl` | Low | Fetch and enumerate .md files from GitHub repos |
| `regenerate` | Medium | Per-layer regeneration (v1.5) |

**Job lifecycle:**

```
Request received → validate + auth → create job in Turso (QUEUED)
    → dispatch to `summarize` queue
    → summarizer completes → dispatch to `tts` queue (if voice requested)
    → TTS completes → dispatch to `bundle` queue
    → bundler assembles .mx + uploads to R2 → job status: COMPLETE
    → notify client via SSE / webhook
```

Retry: 3 attempts with exponential backoff (5s, 30s, 120s). Timeout: 60s summarization, 120s TTS, 30s bundling, 300s repo crawl.

### 5.3 Summarizer Service

**Runtime:** Python 3.12 container (RunPod/Modal serverless GPU)

Pre-processing extracts document structure (headings, word count, code blocks, tables, language) to inform model routing and prompting.

Model routing: files >30K words or premium users → Claude API. Files >15K words → Qwen3-8B. Files <500 words → SmolLM3-3B. Default → Qwen3-8B.

### 5.4 TTS Engine Service

**Runtime:** Python 3.12 container (Fly.io, CPU instance — no GPU needed for Kokoro)

Pipeline: Kokoro inference → pydub normalization → silence trimming → MP3 export → R2 upload.

### 5.5 MX Bundler Service

**Runtime:** Node.js on Cloudflare Workers

Assembles YAML frontmatter + original markdown body, computes SHA-256, uploads to R2, generates hosted URL.

### 5.6 Metadata Store (Turso / LibSQL)

```sql
CREATE TABLE jobs (
    id            TEXT PRIMARY KEY,
    user_id       TEXT,
    status        TEXT NOT NULL,           -- QUEUED | PROCESSING | COMPLETE | FAILED
    input_type    TEXT NOT NULL,           -- file | url | content | repo
    input_hash    TEXT NOT NULL,
    mx_hash       TEXT,
    audio_url     TEXT,
    hosted_url    TEXT,
    model_used    TEXT,
    token_count   INTEGER,
    guidance_json TEXT,                    -- v2: JSON blob of guidance parameters
    created_at    TEXT NOT NULL,
    completed_at  TEXT,
    error_message TEXT
);

CREATE TABLE mx_edits (                   -- v1.5: track layer edits
    id            TEXT PRIMARY KEY,
    mx_hash       TEXT NOT NULL,
    user_id       TEXT NOT NULL,
    layer         TEXT NOT NULL,           -- oneliner | gist | voice_brief | eli5
    action        TEXT NOT NULL,           -- manual_edit | regenerate
    instruction   TEXT,                    -- user instruction for regeneration
    previous_content_hash TEXT,
    new_content_hash TEXT,
    created_at    TEXT NOT NULL
);

CREATE TABLE lens_templates (             -- v2: reusable guidance profiles
    id            TEXT PRIMARY KEY,
    user_id       TEXT NOT NULL,
    name          TEXT NOT NULL,           -- "Executive Brief", "Dev Onboarding"
    guidance_json TEXT NOT NULL,           -- stored guidance parameters
    created_at    TEXT NOT NULL,
    updated_at    TEXT NOT NULL
);

CREATE TABLE users (
    id            TEXT PRIMARY KEY,
    email         TEXT UNIQUE,
    api_key_hash  TEXT,
    tier          TEXT DEFAULT 'free',
    conversions_this_month INTEGER DEFAULT 0,
    created_at    TEXT NOT NULL
);
```

---

## 6. Frontend Architecture

### 6.1 Web Application

**Framework:** React 19 + TypeScript + Vite
**Styling:** Tailwind CSS 4 + Radix UI primitives
**Markdown rendering:** `react-markdown` + `remark-gfm` + `rehype-highlight` + `rehype-slug`
**State management:** Zustand
**Real-time updates:** Server-Sent Events (SSE)
**Hosting:** Cloudflare Pages

**Key UI Components by Version:**

| Component | v1 | v1.5 | v2 |
|-----------|-----|------|-----|
| `MarkdownRenderer` (full GFM rendering) | ✅ | ✅ | ✅ |
| `SidebarTOC` (heading navigation) | ✅ | ✅ | ✅ |
| `LayerPanel` (collapsible layer cards) | ✅ (read-only) | ✅ (editable) | ✅ (editable + variants) |
| `AudioPlayer` (voice brief playback) | ✅ | ✅ | ✅ |
| `FileUploader` (drag-drop, URL, GitHub) | ✅ | ✅ | ✅ |
| `ProgressTracker` (SSE job status) | ✅ | ✅ | ✅ |
| `InlineEditor` (contenteditable layers) | ❌ | ✅ | ✅ |
| `RegenerateButton` (per-layer "try again") | ❌ | ✅ | ✅ |
| `GuidancePanel` (audience, tone, emphasis) | ❌ | ❌ | ✅ |
| `SourceAnnotator` (highlight, strikethrough, notes) | ❌ | ❌ | ✅ |
| `VariantPicker` (choose between AI options) | ❌ | ❌ | ✅ |
| `LensTemplateManager` (save/load guidance profiles) | ❌ | ❌ | ✅ |

**Responsive breakpoints:**

| Breakpoint | Layout |
|-----------|--------|
| Desktop (≥1024px) | Two-column: sidebar (280px fixed) + main content |
| Tablet (768–1023px) | Collapsible sidebar (drawer), full-width content |
| Mobile (<768px) | Bottom sheet for layers, swipeable tabs, full-width content |

**PWA support:** Service worker for offline caching, app manifest, background sync.

### 6.2 Mobile Applications (iOS + Android)

**Framework:** React Native (Expo SDK 52+)

**Rationale:** Maximum code sharing with React web app (~60-70% shared business logic via `@gistmx/core` npm package). Shared: API client, `.mx` parser, Zustand stores, Zod schemas, utilities. Platform-specific: UI components, audio player, file system, push notifications.

**Mobile-Specific Features by Version:**

| Feature | v1 | v1.5 | v2 |
|---------|-----|------|-----|
| View `.mx` files with rendered markdown | ✅ | ✅ | ✅ |
| Audio playback with lock-screen controls | ✅ | ✅ | ✅ |
| Offline cache (SQLite + file system) | ✅ | ✅ | ✅ |
| Share sheet integration | ✅ | ✅ | ✅ |
| Deep linking (`gist.mx/v/{hash}`) | ✅ | ✅ | ✅ |
| Dark mode (system-aware) | ✅ | ✅ | ✅ |
| Inline layer editing | ❌ | ✅ | ✅ |
| Per-layer regeneration | ❌ | ✅ | ✅ |
| Guidance input (card-based flow) | ❌ | ❌ | ✅ |
| Lens template picker | ❌ | ❌ | ✅ |

**Build:** EAS Build (Expo). **Distribution:** TestFlight / Google Play Internal Track → store release. **OTA:** EAS Update for non-native changes.

**Minimum OS:** iOS 16.0+, Android API 26+ (Android 8.0).

---

## 7. API Specification

### 7.1 Authentication

| Tier | Conversions/month | Requests/minute | Max file size | Repo mode | Editing | Lens |
|------|-------------------|-----------------|---------------|-----------|---------|------|
| Anonymous | 3/day | 5/min | 100KB | No | No | No |
| Free | 10/month | 10/min | 500KB | No | Yes | No |
| Pro | Unlimited | 30/min | 5MB | Yes (50 files) | Yes | Yes |
| Team | Unlimited | 60/min | 10MB | Yes (200 files) | Yes | Yes |

### 7.2 Core Endpoints

#### POST /api/v1/convert (v1)

```typescript
interface ConvertRequest {
  file?: File;                          // multipart upload
  url?: string;                         // URL to raw .md file
  content?: string;                     // raw markdown string

  layers?: ('oneliner' | 'gist' | 'voice_brief' | 'eli5')[];
  voice?: boolean;                      // default: true
  voice_id?: string;                    // default: 'af_heart'
  voice_speed?: number;                 // 0.75–2.0, default: 1.0
  format?: 'mx' | 'json';              // default: 'mx'
  webhook_url?: string;
}

interface ConvertResponse {
  job_id: string;
  status: 'QUEUED';
  estimated_seconds: number;
  stream_url: string;
}

interface JobResult {
  job_id: string;
  status: 'COMPLETE';
  mx_url: string;
  hosted_url: string;
  audio_url?: string;
  layers: {
    oneliner: string;
    gist: string;
    voice_brief?: { script: string; duration_seconds: number; };
    eli5: string;
  };
  metadata: {
    model: string;
    token_count: number;
    processing_seconds: number;
    source_hash: string;
  };
}
```

#### PATCH /api/v1/mx/{hash}/layers (v1.5)

```typescript
interface UpdateLayersRequest {
  layers: {
    oneliner?: string;                  // new content (if editing)
    gist?: string;
    voice_brief_script?: string;
    eli5?: string;
  };
  regenerate_voice?: boolean;           // re-generate audio from updated script
}

interface UpdateLayersResponse {
  mx_url: string;                       // updated .mx file URL
  audio_url?: string;                   // new audio URL (if regenerated)
  edit_history: EditEvent[];            // appended edit events
}
```

#### POST /api/v1/mx/{hash}/regenerate (v1.5)

```typescript
interface RegenerateRequest {
  layer: 'oneliner' | 'gist' | 'voice_brief' | 'eli5';
  instruction?: string;                 // e.g., "Make it more technical"
  variants?: number;                    // number of alternatives to generate (1-3, default: 1)
}

interface RegenerateResponse {
  variants: {
    content: string;
    token_count: number;
  }[];
}
```

#### POST /api/v1/convert-guided (v2 — Lens)

```typescript
interface GuidedConvertRequest extends ConvertRequest {
  guidance: {
    author?: string;
    audience?: string;
    key_takeaway?: string;
    tone?: 'technical' | 'casual' | 'persuasive' | 'neutral' | 'excited' | string;
    emphasis?: string[];
    skip?: string[];
    highlights?: {
      section_heading: string;
      note: string;
    }[];
    lens_template_id?: string;          // use saved guidance profile
  };
}
```

#### POST /api/v1/convert-repo (v1)

```typescript
interface ConvertRepoRequest {
  repo_url: string;
  branch?: string;
  path?: string;
  include?: string[];                   // default: ['**/*.md']
  exclude?: string[];                   // default: ['node_modules/**']
  project_summary?: boolean;            // default: true
  max_files?: number;                   // default: 50, max: 200
}
```

---

## 8. CLI Tool

**Package:** `@gistmx/cli`
**Runtime:** Node.js 20+
**Install:** `npm install -g @gistmx/cli` or `npx @gistmx/cli`

```bash
# v1: Convert
gistmx convert README.md                      # → README.mx
gistmx convert README.md --no-voice           # skip audio
gistmx convert README.md --format json        # output JSON
gistmx convert https://raw.githubusercontent.com/.../README.md
gistmx convert ./docs/ --recursive --project-summary
gistmx repo https://github.com/user/repo --path docs/

# v1.5: Edit and regenerate
gistmx edit README.mx --layer oneliner --content "New one-liner text"
gistmx regenerate README.mx --layer eli5 --instruction "Use space analogies"

# v2: Guided conversion (Lens)
gistmx convert README.md --audience "executives" --tone "persuasive" \
    --emphasize "cost,timeline" --skip "implementation details"
gistmx convert README.md --lens-template "executive-brief"

# Utility
gistmx view README.mx                         # pretty-print layers
gistmx view README.mx --layer oneliner
gistmx auth login
```

---

## 9. GitHub Action

```yaml
name: Generate .mx files
on:
  push:
    paths: ['docs/**/*.md', 'README.md']

jobs:
  convert:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: gistmx/convert-action@v1
        with:
          api-key: ${{ secrets.GISTMX_API_KEY }}
          input: 'docs/**/*.md'
          output-dir: 'docs/mx/'
          project-summary: true
          voice: true
          # v2 (future): lens-template: 'executive-brief'
      - uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: 'chore: regenerate .mx files'
```

---

## 10. Infrastructure and Deployment

### 10.1 Service Map

| Service | Runtime | Scaling |
|---------|---------|---------|
| API Gateway | Cloudflare Workers | Auto (global edge) |
| Web App | Cloudflare Pages | Static (global CDN) |
| Task Queue | Upstash Redis | Serverless |
| Summarizer | RunPod Serverless (GPU) | 0→N pods, scale-to-zero |
| TTS | Fly.io (CPU) | 0→3 machines, scale-to-zero |
| MX Bundler | Cloudflare Workers | Auto (global edge) |
| Metadata DB | Turso | Serverless (edge replicas) |
| Object Store | Cloudflare R2 | Serverless |
| Mobile Builds | EAS Build (Expo) | On-demand |

### 10.2 Cost Model (Monthly)

| Component | Low (1K conv/mo) | Medium (5K) | High (20K) |
|-----------|-----------------|-------------|------------|
| Cloudflare (Workers + Pages + R2) | Free | $1.50 | $11 |
| Upstash Redis | Free | $10 | $30 |
| RunPod GPU (A10G) | $5 | $25 | $100 |
| Fly.io CPU (TTS) | $3 | $8 | $25 |
| Turso DB | Free | Free | $29 |
| API fallback (Claude) | $2 | $10 | $40 |
| Domain (gist.mx) | $3 | $3 | $3 |
| Apple Developer (amortized) | $8.25 | $8.25 | $8.25 |
| Google Play (amortized) | $2.08 | $2.08 | $2.08 |
| **Total** | **~$23/mo** | **~$68/mo** | **~$248/mo** |

### 10.3 Deployment Pipeline

```
GitHub Push → CI (lint, type check, test, build)
    → main merge:
        → Deploy Web → Cloudflare Pages
        → Deploy API → Cloudflare Workers
        → Deploy Summarizer → RunPod
        → Deploy TTS → Fly.io
        → DB Migrations → Turso
        → Smoke tests

Mobile (separate):
    → EAS Build → TestFlight / Internal Track → Store release
    → OTA updates via EAS Update
```

---

## 11. Security

- TLS 1.3 in transit, R2 server-side encryption at rest.
- Source `.md` files deleted from workers after conversion. Only `.mx` and audio persist.
- Anonymous conversions auto-deleted after 7 days.
- No user content used for model training. Self-hosted models have no exfiltration path.
- API keys hashed (bcrypt). JWTs expire after 1 hour.
- CORS restricted to `gist.mx` and `*.gist.mx`.
- URL inputs validated (https only, SSRF prevention via internal IP blocklist).
- GitHub repo crawling respects `robots.txt`, rate limited at 5,000 req/hr.

---

## 12. Monitoring

| Layer | Tool | Tracks |
|-------|------|--------|
| Uptime | Cloudflare Health Checks | API + web availability |
| Errors | Sentry | Unhandled exceptions |
| Metrics | Prometheus + Grafana (Fly.io) | Throughput, latency p50/p95/p99, queue depth |
| Logs | Cloudflare Logpush → R2 → Loki | Structured JSON |
| Alerts | PagerDuty | Queue >100, error rate >5%, GPU >90% |
| Cost | Custom dashboard | Daily spend tracking |

---

## 13. Testing

| Layer | Framework | Target |
|-------|-----------|--------|
| Unit (shared logic) | Vitest | 90%+ |
| Unit (API routes) | Vitest + miniflare | 85%+ |
| Unit (summarizer) | pytest | 80%+ |
| Unit (TTS) | pytest | 75%+ |
| Integration | Vitest + Docker Compose | Critical paths |
| E2E (Web) | Playwright | Happy path + error states |
| E2E (Mobile) | Detox | Happy path per platform |
| Load | k6 | 100 concurrent, p99 <30s |
| Model quality | Human-rated golden set (50 .md → .mx pairs) | Quarterly |
| Edit round-trip (v1.5) | Vitest | Edit → save → reload → verify content match |
| Lens output quality (v2) | Human eval (20 guided .mx vs 20 unguided) | Guided output rated higher 70%+ of the time |

---

## 14. Rollout Plan

### Phase 1: Core v1 (Weeks 1–3)
- API gateway, task queue, summarizer service with Qwen3-8B
- Single-file `.md` → `.mx` conversion (all text layers)
- Web UI: upload, paste URL, view results with sidebar
- `.mx` file download and hosted shareable links
- Basic auth (API key, free tier)

### Phase 2: Voice + Polish (Weeks 4–5)
- TTS service with Kokoro
- Audio playback in web UI (speed control, download)
- Mobile-responsive web UI (PWA installable)
- CLI tool v1.0 (`@gistmx/cli`)

### Phase 3: Multi-File + Mobile (Weeks 6–8)
- GitHub repo crawling and multi-file conversion
- Project-level meta `.mx` generation
- React Native app (iOS + Android): view, listen, offline cache
- GitHub Action v1.0

### Phase 4: v1 Launch (Weeks 9–10)
- Model quality evaluation and tuning
- Mobile app TestFlight / internal track
- Public API documentation (gist.mx/docs)
- Product Hunt launch
- App Store / Google Play submission

### Phase 5: v1.5 Inline Editing (Weeks 11–13)
- Inline contenteditable for all layers in web viewer
- Per-layer "regenerate" with optional instruction
- `manually_edited` and `edit_history` in `.mx` files
- PATCH and regenerate API endpoints
- Mobile editing support
- CLI `edit` and `regenerate` commands

### Phase 6: v2 Lens (Weeks 16–22)
- Guidance input UI (audience, tone, emphasis, skip, key takeaway)
- Source annotation (highlight, strikethrough, margin notes)
- Variant generation (2-3 options per layer, user picks)
- Layer locking (approve individual layers, regenerate others)
- Lens templates (save/load/share guidance profiles)
- Author attribution in `.mx` files
- Guided conversion API endpoint
- CLI `--audience`, `--tone`, `--emphasize`, `--lens-template` flags
- Mobile: card-based guidance flow

---

## Appendix A: Dependency Inventory

### Backend
| Package | Purpose | License |
|---------|---------|---------|
| hono 4.x | API framework (Workers) | MIT |
| bullmq 5.x | Task queue | MIT |
| vllm 0.8.x | LLM inference server | Apache 2.0 |
| kokoro 1.x | TTS engine | Apache 2.0 |
| pydub 0.25.x | Audio processing | MIT |
| zod 3.x | Schema validation | MIT |
| @upstash/redis 1.x | Redis client | MIT |
| @libsql/client 0.x | Turso DB client | MIT |

### Frontend (Web)
| Package | Purpose | License |
|---------|---------|---------|
| react 19.x | UI framework | MIT |
| react-markdown 9.x | Markdown rendering | MIT |
| remark-gfm 4.x | GFM support | MIT |
| rehype-highlight 7.x | Code highlighting | MIT |
| zustand 5.x | State management | MIT |
| @radix-ui/* | Accessible primitives | MIT |
| tailwindcss 4.x | Utility CSS | MIT |

### Frontend (Mobile)
| Package | Purpose | License |
|---------|---------|---------|
| react-native 0.76+ | Mobile framework | MIT |
| expo 52+ | Build/deploy | MIT |
| expo-av | Audio playback | MIT |
| expo-file-system | Local storage | MIT |
| expo-sqlite | Offline DB | MIT |
| nativewind 4.x | Tailwind for RN | MIT |

### AI Models
| Model | Purpose | License |
|-------|---------|---------|
| Qwen3-8B-Instruct-AWQ | Primary summarization | Apache 2.0 |
| SmolLM3-3B | Lightweight fallback | Apache 2.0 |
| Kokoro-82M v1.0 | Primary TTS | Apache 2.0 |
| Chatterbox | Premium TTS (future) | MIT |

---

## Appendix B: Competitive Moat Analysis

| Moat | Strength | Why |
|------|----------|-----|
| `.mx` file format | Strong | Network effects: as more tools read/write `.mx`, the format becomes a standard. First-mover in "comprehension layer for markdown." |
| Multi-layer comprehension | Medium | Easy to replicate technically, but the specific layer design (voice script ≠ summary, ELI5 ≠ short summary) requires opinionated product thinking. |
| Voice as first-class | Medium | Kokoro is open-source; anyone can add TTS. But purpose-written voice scripts (not TTS-on-summary) require prompt engineering that's hard to match without investment. |
| Project-level synthesis | Strong | Cross-file relationship analysis, redundancy elimination, hierarchy inference — this is significantly harder than single-file summarization. |
| Lens / guided authoring (v2) | Very strong | Transforms output from commodity auto-summary into human-guided briefing. The guidance IS the content. No one else has this. |
| Hosted viewer + share links | Medium | Easy to build, but gist.mx has the natural domain and URL structure (`gist.mx/v/{hash}`). |
| Ecosystem (CLI + Action + API + mobile) | Strong | The breadth of distribution surfaces makes it the default way to produce `.mx`. Higher switching cost. |
