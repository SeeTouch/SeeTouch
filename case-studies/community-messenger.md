# Women's Community Messenger

> A women-only community messenger with local-first, offline-capable real-time chat. *(Private client project — shared without name or visuals under NDA.)*

<kbd>Role: Solo developer</kbd> · <kbd>Flutter/Dart + Supabase</kbd> · <kbd>Mar 2026 – Jun 2026</kbd> · <kbd>Status: MVP (TestFlight)</kbd>

## Overview

A mobile messenger built for a women's community — real-time direct messaging with an Apple-first iOS experience and a stable Android fallback. The hard problem is not the chat UI; it is making messaging feel instant and reliable on flaky mobile networks. The app solves this with a local-first architecture: the UI always reads from an on-device reactive database, while a custom sync engine reconciles that cache with a managed Postgres backend over realtime subscriptions. Releases ship through an automated iOS pipeline.

## What it does

- Real-time one-to-one messaging with optimistic send, delivery and read receipts, and draft persistence
- A chat-request gate so conversations only open once the recipient accepts (anti-harassment by design)
- User discovery with filters (interests, languages, location) backed by a large seeded geo catalog
- Profile setup with avatars and photos in cloud storage
- Push notifications with deep-linking into the right conversation, plus app badge counts
- Admin verification and moderation tooling (blocks, reports, account deletion)
- Multi-language localization
- Offline-first behavior: messages queue locally and reconcile automatically when connectivity returns

## Tech stack

**Frontend/App:** Flutter (Dart), Riverpod for state management, go_router for navigation, a custom wrapper-based UI kit over Cupertino/Material for iOS-native large-title pages, typed i18n.
**Backend/Services:** Supabase — Postgres with row-level security, Realtime, Auth, and Storage. A Deno/TypeScript edge function dispatches push via Firebase Cloud Messaging.
**Data/Infra:** Drift (SQLite) as the local reactive cache and source of truth for the UI; secure storage for session artifacts; crash reporting; local + remote push notifications.
**Tooling/Tests:** flutter_test + mocktail, strict lint rules, build_runner/freezed/json_serializable codegen, GitHub Actions for iOS delivery, Architecture Decision Records (Michael Nygard format).

## Architecture & engineering highlights

- **Local-first, single-direction reads.** The UI never subscribes to the network. Every screen reads from Drift via reactive streams; the sync engine mirrors realtime events into Drift through idempotent UPSERTs that respect server timestamps. This removes per-screen network fetches and eliminates loading flicker.
- **Custom sync engine (~3,000+ LOC across sync, realtime, and outbox modules).** Handles bootstrap sync split into critical vs. deferred phases, optimistic updates with an outbox/retry pattern, realtime merge, and per-domain sync cursors. Bootstrap atomicity is enforced so anchor rows and their referenced profiles land in the same phase, avoiding a fallback flash on first paint.
- **Keyset pagination for message history.** Message history pages use a `(created_at, id)` keyset cursor with UUIDv7 ids for a stable, monotonic tie-break, extracted as a pure, unit-tested function so the wire format is verified without a live client.
- **Wrapper-enforced UI architecture.** All shell integrations (scaffold, top/bottom bars, sheets, dialogs, sliver pages) go through app-owned wrappers, so third-party or experimental UI packages stay replaceable and screens never depend on them directly.
- **Unified conversations model.** Direct chats are conversations with two members; the same base schema is designed to extend to groups, community rooms, and events without a rewrite.
- **Documented decision trail.** Significant choices (a cross-platform framework spike, deferring a managed-sync migration to post-MVP, cloud now / self-host at scale) are captured as ADRs rather than tribal knowledge.

## Engineering challenges solved

- **Reliable optimistic messaging over unreliable networks.** Approach: optimistic insert into Drift, an outbox entry, server acknowledgement, then reconciliation. On reconcile failure the row stays marked `pending` and retries rather than being silently dropped. Result: messages appear instantly, survive offline gaps, and never vanish.
- **Realtime correctness without UI coupling to the backend.** Approach: a single ingestion path where realtime events are merged into Drift idempotently by primary key, with the UI subscribing only to Drift streams. Result: one consistent source of truth and predictable behavior across bootstrap, foreground refresh, and live catch-up.
- **Honest security posture.** Phase 1 is explicitly cloud-secured (TLS in transit, encryption at rest, RLS on exposed tables, secure local session storage) — not end-to-end encryption — and the project rules forbid claiming E2EE anywhere until it is actually implemented. RLS performance and free-tier load were hardened in dedicated migrations.
- **Automated, signing-safe iOS delivery.** A tagged release triggers a GitHub Actions workflow that builds with Xcode and uploads to the Apple beta channel via the App Store Connect API, with a documented local fallback script — keeping manual signing under control while making releases repeatable.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-03-05 → 2026-06-22 |
| Commits | 723 |
| Active development days | 49 |
| Lines of code | ~51,300 Dart (hand-written, excl. ~17K generated) + ~6,500 SQL migrations + ~1,300 TS edge function |
| Languages | Dart (app), SQL (Postgres/RLS migrations), TypeScript (Deno edge function) |
| Database migrations | 87 SQL migration files |
| Tests | 478 test cases across 69 files (flutter_test + mocktail) |

*LOC note: the SQL figure excludes a large generated geo-catalog data seed; only hand-written schema, RLS, RPC, and trigger migrations are counted.*

## Screenshots

Not shared publicly (NDA). Available privately on request.
