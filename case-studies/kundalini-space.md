# Kundalini Space

> A Telegram Mini App platform delivering subscription-gated practice videos to a wellness community.

<kbd>Role: Solo developer</kbd> · <kbd>Python · Flutter · FastAPI · PostgreSQL</kbd> · <kbd>Nov 2025 – Mar 2026</kbd> · <kbd>Status: MVP / Prototype</kbd>

## Overview

Kundalini Space is a content-delivery platform for a Kundalini yoga teaching practice. It turns a back-catalog of recorded Zoom classes into a structured, subscription-gated video library accessible through a Telegram Mini App. The system spans three concerns: an offline media-processing pipeline that prepares the raw recordings, a FastAPI backend that serves a catalog with access control, and a Flutter client embedded inside Telegram. The project is an MVP — several layers are functional, while orchestration of payments and notifications was scoped as low-code (n8n/Flowise) workflows rather than fully implemented in code.

## What it does

- Ingests raw class recordings and produces a clean catalog: extracts and normalizes audio, transcribes speech (local Whisper or OpenAI API), generates AI summaries, thumbnails, and metadata.
- Serves a browsable catalog of practices and content over a REST API, with per-user product access and a time-limited demo mode.
- Authenticates Telegram users by validating WebApp `initData` (HMAC signature check) instead of a separate login flow.
- Tracks viewing history and per-user statistics; exposes an admin surface for seeding content, granting access, and inspecting users.
- Streams video from object storage (Cloudflare R2) with thumbnails synced alongside.

## Tech stack

**Frontend/App:** Flutter (Dart) Telegram Mini App targeting Web, iOS, and Android — `telegram_web_app`, `go_router`, `flutter_riverpod`, `dio`, `chewie`/`video_player`, Material 3. A prior React/Vite + TypeScript client is retained as a superseded iteration.
**Backend/Services:** FastAPI (Python) with ~19 REST endpoints, SQLAlchemy ORM, Alembic migrations, Pydantic schemas, custom Telegram `initData` auth.
**Data/Infra:** PostgreSQL; Cloudflare R2 (S3-compatible) for media; Docker Compose for dev/local/prod; Traefik reverse proxy with ACME/Let's Encrypt; Supabase schema + RLS migrations for an alternate data layer.
**Media pipeline:** OpenAI Whisper, pydub, ffmpeg-python, OpenAI API for summarization.
**Orchestration (specified):** n8n for webhooks/cron, Flowise for AI agent logic, Stripe and Zoom integrations — documented as workflow specs.
**Tooling/Tests:** flutter_lints, eslint; no automated test suite present.

## Architecture & engineering highlights

- **Separation of pipeline and serving layers.** Heavy, non-interactive media processing (`processor.py`, `main.py`) runs offline and writes a catalog; the API layer only serves the prepared artifacts. This keeps the request path light and decouples expensive ML work from user traffic.
- **Modular processing stages.** The pipeline is composed of discrete classes — `AudioExtractor`, `Transcriber`, `Summarizer`, `ThumbnailGenerator`, `MetadataGenerator` — each with a single responsibility, allowing transcription to switch between a local Whisper model and the OpenAI API via configuration.
- **Telegram-native authentication.** Rather than a custom login, the backend verifies the cryptographic `initData` signature Telegram sends to Mini Apps, using HMAC with a key derived from the bot token. This is the correct, documented trust model for TMAs and avoids storing passwords.
- **Access-control data model.** Eight related entities (`User`, `Product`, `UserProductAccess`, `Content`, `Tag`, `UserHistory`, `Purchase`, `UserPersonalAccess`) model subscription products, per-user grants, personal one-off access, and viewing history, including a demo-access mechanism.
- **Production-shaped deployment.** Distinct Compose files for dev, local, and prod; Traefik handles routing and TLS; documented fixes for Traefik v3 routing, cert resolvers, healthchecks, and a concurrent table-creation race condition.

## Engineering challenges solved

- **Turning unstructured Zoom recordings into a searchable library.** Raw recordings carry no titles, summaries, or thumbnails. The pipeline normalizes audio levels, transcribes with Whisper, and uses an LLM to generate human-readable summaries and metadata, producing catalog entries without manual tagging of every video.
- **Embedding a real app inside Telegram.** Running Flutter as a Mini App required disciplined viewport handling — disabling vertical swipes so upward gestures don't close the app, expanding to full height, and suppressing browser overscroll — plus theme sync to Telegram's params and adaptive Cupertino/Material rendering per platform.
- **Production routing and concurrency.** Several commits address real deployment failures: correcting the API path prefix behind Traefik, fixing the cert resolver and Docker network attachment, and resolving a race where concurrent table creation collided on startup.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2025-11-23 → 2026-03-05 |
| Commits | 15 |
| Active development days | 4 |
| Lines of code | ~8,400 authored (excl. deps, lockfiles, and the superseded React client) |
| Languages | Python ~4,140; JSX ~2,110; SQL ~1,540; Dart ~960; TS/JS ~780; YAML (infra) ~475; CSS ~140 |
| Tests | None (manual verification via Chrome and iOS Simulator) |

## Screenshots

Available on request.
