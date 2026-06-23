# TopicDesk

> An omnichannel support desk and CRM that runs entirely inside Telegram forum topics.

<kbd>Role: Solo developer</kbd> · <kbd>TypeScript / Deno / Supabase Edge Functions</kbd> · <kbd>Mar–Apr 2026</kbd> · <kbd>Status: MVP (deployed)</kbd>

## Overview

Small businesses — coaches, consultants, micro-shops — juggle customer conversations across WhatsApp, Telegram, Instagram, and email, while traditional CRMs are too heavy and expensive for them. TopicDesk turns a single Telegram group into a unified support desk: each customer becomes a dedicated forum topic, and the manager works entirely from the Telegram app they already use. There is no separate dashboard to learn — status changes, CRM cards, bookings, and broadcasts are all driven through native Telegram primitives (topic names, pinned messages, inline keyboards, hashtags).

The system runs as a stateless webhook on Supabase Edge Functions — no servers to maintain, no polling — with per-workspace bot tokens, scheduled jobs for reminders and subscription lifecycle, and a GDPR data-export/delete API.

## What it does

- Routes inbound customer messages into per-customer forum topics: new contacts trigger topic creation, a pinned CRM card, and message forwarding; returning contacts resolve to their existing topic.
- Manages CRM state through the interface itself — renaming a topic with an emoji prefix changes the contact's status, and hashtags in a topic are parsed into tags.
- Maintains a pinned, editable CRM card per contact (name, phone, plan, balance, notes, activity sparkline) updated via inline keyboards and ForceReply prompts.
- Handles bookings, schedules, services, and subscription plans with an in-chat calendar and time-picker built from inline keyboards.
- Sends broadcasts to active contacts or to a specific tag segment from the group's General topic.
- Runs scheduled jobs (via pg_cron + Edge Functions) for renewal reminders, no-show marking, subscription expiry, deadline alerts, recording delivery, and a daily digest.
- Lets each workspace connect its own customer-facing bot, with tokens stored encrypted in Supabase Vault rather than in the database.

## Tech stack

**Bot/App:** grammY (TypeScript on Deno), Hono for HTTP APIs
**Backend/Services:** Supabase Edge Functions (telegram-webhook, events-api, admin-api, and six cron functions)
**Data/Infra:** PostgreSQL via Supabase, Row-Level Security, Supabase Vault for secrets, pg_cron for scheduling, pgvector reserved for future RAG
**Tooling/Tests:** Deno test runner, GitHub Actions CI/CD deploying migrations and functions to Supabase on push to main

## Architecture & engineering highlights

- **Stateless serverless bot.** All business logic lives in a single `telegram-webhook` Edge Function organized into `handlers/`, `services/`, and `middleware/` layers — no long-running process, no polling. The webhook always returns 200 to Telegram and funnels errors through `bot.catch()` into a `bot_errors` table, so a failing handler never blocks the update stream.
- **Account → Workspace tenancy.** A clean two-level model (`accounts` = Telegram user + plan, `workspaces` = 1:N) scopes every domain table by `workspace_id`. A `workspace-resolver` middleware establishes tenancy on each update, and a `plan-enforcer` middleware gates features by subscription plan.
- **Secrets via Vault, never the DB.** Per-workspace bot tokens are encrypted into Supabase Vault through RPC (`encrypt_secret`), with only opaque UUID references stored in application tables — there are no raw tokens at rest.
- **Interface-as-CRM.** Rather than a separate admin UI, the product reuses Telegram's own primitives — emoji-prefixed topic titles encode status, pinned messages render the live CRM card, and inline-keyboard calendar/time-picker widgets handle scheduling — keeping the operator experience zero-learning.
- **CI/CD as the only deploy path.** Migrations and functions deploy exclusively through GitHub Actions on push to main; direct DDL via tooling is prohibited by convention, keeping schema changes reviewable and reproducible.

## Engineering challenges solved

- **Reliable routing under Telegram's webhook contract.** Telegram retries on any non-200 response, which can amplify a single bad update into a storm. A dedicated `message-router` service plus a catch-all error boundary guarantee a 200 acknowledgement while still capturing the failure for diagnosis, decoupling delivery reliability from handler correctness.
- **Rich UI inside a chat client.** Calendars, time pickers, paginated menus, and editable cards were all built from inline-keyboard state machines and ForceReply flows, with dedicated unit tests for the keyboard and time-relative rendering logic to keep the stateless rendering deterministic.
- **Scheduling without a scheduler service.** Recurring work (reminders, expiry, no-show, digests) is driven by pg_cron triggering thin Edge Functions, avoiding any always-on worker while keeping the entire system serverless.
- **Migration drift management.** Aligning a locally evolving schema with the remote Supabase project required a disciplined baseline-and-numbered-migration scheme (initial schema through the Phase 5 cron jobs, plus baseline stubs to reconcile with remote state).

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-03-05 → 2026-04-29 |
| Commits | 230 |
| Active development days | 8 |
| Lines of code | ~43,500 (excl. deps): ~37,450 TypeScript, ~6,058 SQL |
| Languages | TypeScript ~86%, SQL ~14% |
| Tests | ~117 cases across 6 Deno test files (keyboards, time/relative-time, sparkline, admin dashboard) |

## Screenshots

UI prototypes exist as HTML mockups under `.superpowers/brainstorm/` (calendar widget, time picker, client menu, services CRUD, topic profile). Product screenshots: available on request.
