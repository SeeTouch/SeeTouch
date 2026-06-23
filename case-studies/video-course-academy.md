# Online Video-Course Academy

> A subscription video-course platform for online education, delivered across web and mobile. *(Private client — shared without brand name.)*

<kbd>Role: Solo developer</kbd> · <kbd>Flutter + Supabase + Next.js</kbd> · <kbd>2026-03-28 → 2026-04-27</kbd> · <kbd>Status: MVP</kbd>

## Overview

This is an online video-course platform built around subscriptions and per-course purchases. A student-facing app (web, iOS, Android) handles browsing, playback, and progress tracking; a web admin panel manages courses, lessons, media, and users; and a separate marketing site converts visitors. The system is designed for a multi-language audience and includes the schema foundations for an AI knowledge base (transcription and semantic search over course content).

This is a standalone product, not an evolution of the separate omnichannel messaging CRM described in another case study — a different project that happens to share a similar Flutter + Supabase stack.

## What it does

- Lets students browse a course catalog, watch HLS video lessons, and resume where they left off with saved progress.
- Gates paid and subscription-only content behind server-side access checks, showing lock icons and upgrade calls-to-action.
- Gives admins a panel to create and edit courses, modules, and lessons, and to upload media (video, PDF, audio, images) directly to object storage.
- Presents a marketing landing page (hero, features, courses, pricing, testimonials, about) that pulls live data from the backend.
- Supports multiple interface languages (Russian, English, Spanish, German) at the data-model level.

## Tech stack

**Frontend/App:** Flutter 3.38+ / Dart 3.10+ (student app and admin panel); Riverpod 2.6 for state, GoRouter 14.8 for routing, Freezed 3.0 for immutable models; `video_player` + `chewie` with HLS support on web. Next.js 16 / React 19 / Tailwind 4 / Framer Motion for the marketing landing page.
**Backend/Services:** Supabase (Postgres, Auth with PKCE, Realtime, Storage, Edge Functions). A Deno edge function issues presigned upload URLs to Cloudflare R2. Designed integrations for Deepgram (transcription) and Trigger.dev (AI pipeline orchestration).
**Data/Infra:** PostgreSQL with 17 tables, `pgvector` for transcript embeddings, `pg_trgm` for search, and row-level security throughout. Sentry for error tracking.
**Tooling/Tests:** Melos 7.5 monorepo (Dart workspaces), `build_runner` codegen, a Makefile and Melos scripts for setup/run/codegen/analyze, and a GitHub Actions CI workflow.

## Architecture & engineering highlights

- **Three apps, one shared core.** A Melos monorepo hosts the student app, admin panel, and a shared-core package holding Freezed models, Supabase table constants, the auth client, and a sealed `AppException` hierarchy — so both Flutter apps speak the same data contracts without duplication.
- **Feature-first structure.** Each feature is organized into `presentation/`, `providers/`, and `data/` layers, keeping UI, state, and repositories cleanly separated across both apps.
- **Server-side access control.** Course access is never decided on the client. The app calls a Postgres RPC (`has_course_access`) and free courses short-circuit without a round trip; everything else is authorized in the database and enforced by RLS.
- **Secrets stay server-side.** The service-role key and storage credentials live only in edge-function environment variables; the Flutter clients receive configuration via `--dart-define` and only ever hold the public anon key.
- **DB-driven JSON mapping.** camelCase Dart fields map to snake_case Postgres columns via a centralized `build.yaml` field-rename rule rather than scattered annotations.

## Engineering challenges solved

- **Direct-to-storage uploads without exposing credentials.** Large media must not pass through the app server, but R2 credentials must never reach the browser. The solution is an edge function that verifies the caller's JWT, confirms the `admin` role, then hand-rolls an AWS Signature V4 presigned PUT URL (HMAC-SHA256 signing chain, canonical request, `UNSIGNED-PAYLOAD`) so the admin panel uploads straight to Cloudflare R2 under a structured key layout (`videos/`, `documents/`, `audio/`, etc.).
- **Resumable web video playback.** Web HLS playback needed reliable resume and auto-complete. Lessons fetch their video URL from a `lesson_media` lookup and persist progress so students continue from their last position, with completion recorded back to the progress tables.
- **AI knowledge base, designed up front.** The schema reserves `video_transcripts`, `transcript_embeddings`, and a documented Deepgram → Trigger.dev → pgvector pipeline, so retrieval-augmented features can be added without reshaping the data model.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-03-28 → 2026-04-27 |
| Commits | 36 |
| Active development days | 3 |
| Lines of code | ~18,400 (excl. deps & generated) |
| Languages | Dart ~16,900 (81 files), TypeScript/TSX ~1,500 (incl. edge function), SQL ~610 |
| Tests | None present (CI configured; test harness in place) |

Notes: Dart total excludes 12 generated `*.g.dart` / `*.freezed.dart` files. The TypeScript figure covers the Next.js landing page (1,347 lines across 15 files) plus the Supabase edge function (187 lines). The Postgres schema spans 17 tables with 37 RLS policies.

## Screenshots

No screenshot assets are committed to the repository (only app launcher icons and SVG placeholders). Available on request.
