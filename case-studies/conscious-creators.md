# Conscious Creators

> Omnichannel messaging CRM that unifies Telegram, WhatsApp and email into one inbox for a yoga studio.

<kbd>Role: Solo developer</kbd> · <kbd>Flutter / Dart + Supabase</kbd> · <kbd>2026-02-22 → 2026-03-15</kbd> · <kbd>Status: MVP</kbd>

## Overview

Conscious Creators is a communication hub for a yoga business whose teachers field student messages scattered across Telegram, WhatsApp and email. The product collapses those channels into a single conversation inbox, so a teacher answers a student in one app regardless of where the message originated. It is built as a Flutter monorepo on top of a serverless Supabase backend — PostgreSQL with row-level security, Edge Functions as the messaging switchboard, and Realtime for live updates. The MVP delivers end-to-end Telegram messaging (inbound and outbound), offline-first local storage, and an authenticated, themed mobile client.

## What it does

- Receives inbound messages from Telegram via a webhook gateway and stores them in a normalized `messages` table, independent of source platform.
- Sends outbound replies back to the originating platform through a dedicated Edge Function calling the Telegram Bot API.
- Presents a unified chat list with per-conversation unread badges, platform icons, and online/offline presence.
- Renders a real-time chat room with message bubbles, delivery status, and live updates over Supabase Realtime.
- Authenticates teachers via email/password and OTP, and supports a Telegram Mini App auth bridge for the in-Telegram client.
- Works offline-first: messages are cached in a local SQLite database (Drift) and reconciled with the backend.
- Adapts layout for phone and iPad and supports light/dark theming and localization.

## Tech stack

**Frontend/App:** Flutter (Dart), Riverpod for state management, go_router for navigation, Drift over SQLite for the local database, FlexColorScheme theming, Google Fonts (DM Sans), `wolt_responsive_layout_grid` for adaptive layout, `slang` for i18n, `pinput` and `infinite_scroll_pagination` for UI.
**Backend/Services:** Supabase — PostgreSQL, Auth (email/password + OTP), Realtime, and TypeScript Edge Functions (`omni-receiver`, `send-message`, `notify-orchestrator`) running on Deno.
**Data/Infra:** PostgreSQL with row-level security, 14 SQL migrations, generated Dart/TypeScript types from the schema (supadart), Telegram Bot API integration.
**Tooling/Tests:** Melos-managed monorepo (apps + shared packages), GitHub Actions CI/CD for migrations and Edge Function deploys, Flutter widget tests, `build_runner` codegen for Riverpod and Drift.

## Architecture & engineering highlights

- **Serverless omnichannel switchboard.** Rather than running a traditional server, all inbound traffic from every platform hits a single `omni-receiver` Edge Function that verifies the source, normalizes the payload into a common message shape, and writes it to one `messages` table. Adding WhatsApp or email later means extending the receiver, not redesigning the data model.
- **Monorepo with shared packages.** A Melos workspace separates concerns into apps (`mobile`, `miniapp`, plus planned `dashboard` and `landing`) and reusable packages (`db-types`, `integrations`, `ai-agents`, `background-jobs`). Database types are generated from the live schema and shared across Dart and TypeScript so the client and Edge Functions stay in sync with the source of truth.
- **Offline-first sync.** The mobile client stores conversations in a local Drift/SQLite database and reconciles with Supabase, so the inbox is usable without connectivity. Migrations were deliberately reworked from destructive to incremental to preserve local data across schema changes.
- **Security at the data layer.** Authorization lives in PostgreSQL row-level security policies rather than application code; several migrations specifically resolve RLS recursion issues on the `channels`, `channel_participants`, and `messages` tables — a non-obvious failure mode when policies reference each other.
- **CI/CD as code.** A GitHub Actions pipeline runs Flutter analysis and ships database migrations and Edge Functions on merge, with path filters so docs-only changes skip the deploy.

## Engineering challenges solved

- **RLS policy recursion.** Self-referential row-level security policies on the messaging tables caused infinite-recursion errors at query time. Resolved through dedicated migrations that restructured the policies and introduced auth helper functions, keeping authorization enforced in the database without circular evaluation.
- **Platform-agnostic message model.** Different messengers expose wildly different payloads. The solution was a normalization step in the Edge Function layer that maps every source into a single canonical record (sender, source platform, content, metadata), which is what makes the unified inbox possible and the WhatsApp/email roadmap incremental.
- **Non-destructive local migrations.** Early Drift migrations wiped local data on schema change. They were migrated to incremental upgrades and the storage layer moved to `drift_flutter`, preserving offline state across app updates.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-02-22 → 2026-03-15 |
| Commits | 78 |
| Active development days | 7 |
| Lines of code | ~13,000 (excl. deps): ~7,100 hand-written Dart, ~4,500 generated Dart types, ~2,900 TypeScript (Edge Functions + types), ~3,300 SQL (migrations + seed) |
| Languages | Dart (primary), TypeScript (Edge Functions), SQL (schema/RLS) |
| Tests | 3 Flutter widget/unit test files (`test`/`testWidgets`) |

## Screenshots

- `docs/screenshots/IMG_5B3B29EFCE3E-1.jpeg`
- `docs/screenshots/whatsapp chat room.PNG`

Additional UI captures available on request.
