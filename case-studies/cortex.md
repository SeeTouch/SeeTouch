# Cortex

> Turns unstructured PDFs and scanned books into RAG-ready knowledge bases using Gemini vision.

<kbd>Role: Solo developer</kbd> · <kbd>React 19 + TypeScript + Google GenAI</kbd> · <kbd>2026-03 → 2026-05</kbd> · <kbd>Status: MVP</kbd>

## Overview

Cortex is a client-side web application that converts scanned books and unstructured PDFs into structured, AI-enriched knowledge bases optimized for Retrieval Augmented Generation. It addresses a concrete problem in building RAG pipelines: raw documents — especially scanned, multi-column, table-heavy material such as medical or technical reference books — produce poor retrieval quality without careful page parsing, semantic chunking, and metadata enrichment. Cortex runs the entire pipeline in the browser, from PDF rendering through Gemini vision parsing to a final exportable set of RAG chunks complete with hypothetical questions, keywords, and a knowledge graph. It is domain-agnostic: the same engine handles medical, legal, technical, or general documents through configurable domain profiles.

## What it does

- Imports PDFs and images, rendering each page client-side (PDF.js) into images for vision parsing.
- Parses each page with Gemini under a strict transcription contract (verbatim text, literal numbers and units, GitHub-flavored tables, structured-JSON output) and emits per-page diagnostic warnings such as low image quality or truncated tables.
- Runs a multi-pass semantic chunking engine (cleaning, zone removal, physical stitching, table detection, hierarchy-aware splitting, greedy merge, recursive split) that keeps small tables whole and splits large ones by row while preserving headers.
- Enriches each chunk with Gemini: hypothetical questions (HyDE), keywords, dense summaries, and an extracted knowledge graph of entities and relationships constrained to the active domain profile.
- Extracts and crops visual assets (figures, tables) from page images and packages them into a downloadable ZIP.
- Extracts a Table of Contents, validates page sequence, and lets the user edit markdown with a live preview.
- Saves and loads self-contained project files and exports the final chunk set as JSON or ZIP.

## Tech stack

**Frontend/App:** React 19, TypeScript (strict), Vite 6, Tailwind CSS 3, react-markdown with remark-gfm and remark-breaks
**AI/Services:** Google Gemini via `@google/genai` — vision page parsing, TOC extraction, and chunk enrichment with structured response schemas
**Data/Infra:** Fully client-side — no backend; PDF.js for rendering, JSZip for asset packaging, localStorage for settings, Blob URLs for page image memory management
**Tooling/Tests:** Vitest, @testing-library/react, jsdom, ESLint 8 with typescript-eslint, Prettier

## Architecture & engineering highlights

- **Layered, browser-only pipeline.** A clean service layer separates concerns: `pdfService` (render), `geminiService` (AI calls), `chunkingService` (semantic chunking), `graphService` (entity normalization and graph aggregation), `imageService` (asset cropping), and `pricingService` (token cost accounting). Business logic was extracted from `App.tsx` into focused custom hooks (`usePages`, `useProcessing`, `useFileImport`, `useProjectIO`, `useKeyboardNav`), leaving the root component as an orchestrator.
- **Domain-agnostic prompting.** All AI behavior is parametrized through a `DomainProfile` (entity types, relation types, parsing and enrichment hints, language hints). Prompt builders compose instructions from the active profile, so medical, legal, technical, and general presets reuse the same engine with no branching in the model code.
- **Structured-output discipline.** Every Gemini call declares a JSON response schema and a fidelity directive that forbids paraphrasing, rounding, or unit conversion — critical for reference material where "1000 mg" must never become "1 g". Knowledge-graph extraction is constrained to the profile's allowed entity and relation types, falling back to `OTHER`/`RELATED_TO` rather than inventing categories.
- **Concurrent processing with backpressure.** A configurable worker pool drives page parsing with shared-index task stealing, an `AbortController` for instant cancellation, randomized inter-request delays, and an auto-stop that halts the run when the API reports quota exhaustion.

## Engineering challenges solved

- **Reliable extraction from a noisy AI endpoint.** Vision models return empty, malformed, or partial JSON under load. The parsing path wraps each call in retry-with-exponential-backoff plus jitter, validates required fields before accepting a result, races the request against the abort signal so cancellation is immediate, and distinguishes retryable (429/503/quota) from terminal failures.
- **Faithful chunking of table-heavy documents.** Standard recursive splitters destroy tables and merge unrelated sections. The chunking engine adds a dedicated table-detection pass that recognizes markdown pipe tables, keeps small ones intact, and splits oversized ones by row while re-emitting the header — with diagnostic warnings when a header is too large to preserve cleanly.
- **Idempotent, rate-limited enrichment.** Enrichment hashes chunk content (SHA-256, with a non-secure-context fallback) and skips chunks whose hash and enriched state are unchanged, avoiding redundant paid API calls. The knowledge graph is then normalized across the whole project: entity IDs are slugified, types validated against the domain profile, and duplicates merged with majority voting on canonical names and aliases.
- **Browser memory management.** Multi-hundred-page PDFs are handled by storing pages as Blob URLs rather than base64, freeing canvases after render and destroying the PDF document object, with explicit Blob URL revocation on delete and project load, and conversion to base64 only at save time.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-03-05 → 2026-05-08 |
| Commits | 78 |
| Active development days | 5 |
| Lines of code | ~7,100 application (TS/TSX, excl. tests) + ~2,000 tests; ~9,150 total (excl. deps) |
| Languages | TypeScript/TSX (TSX ~4,000, TS ~3,060), with Tailwind/HTML config |
| Tests | 128 cases across 10 Vitest files (unit, hook, and integration suites) |

## Screenshots

Available on request. (The repository contains UI components and domain-preset assets but no committed screenshots; large fixture documents are intentionally gitignored.)
