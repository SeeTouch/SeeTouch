# Yoga Timer

> iOS-first Flutter app for guided yoga and meditation, with presets, audio-backed sessions, and crash-safe recovery.

<kbd>Role: Solo developer</kbd> · <kbd>Flutter / Dart</kbd> · <kbd>2026-03-28 → 2026-05-06</kbd> · <kbd>Status: MVP</kbd>

## Overview

Yoga Timer is a guided practice app for yoga and meditation sessions. A practitioner composes a quick timer or saved preset, runs an interval-based session with ambient audio, a 60 BPM metronome, and transition cues, and the app keeps running correctly through phone calls, headphone unplugs, and app backgrounding. The engineering focus is a clean separation between session business rules, wall-clock timing, audio playback, and persistence, so each concern can be tested and reasoned about in isolation. The result is a working iOS/Android application, not a scaffold: persisted presets, a shared session runtime, an adaptive iOS-first UI, and a project-owned native iOS sheet plugin.

## What it does

- Composes quick timers inline (duration, label, metronome toggle, ambient sound, end sound) and launches them directly from the Practice tab.
- Stores reusable presets with ordered intervals, including optional overlap windows between intervals, persisted locally.
- Runs interval sessions with phase transitions, transition/completion bells, ambient loops, and a fixed-tempo metronome.
- Minimizes an active session to a card and reopens it later; tracks mixed recent launches across quick timers and presets.
- Recovers an in-progress session after backgrounding or interruption, restoring exact elapsed time, current interval, and paused/running state.
- Adapts the UI to platform conventions (Cupertino on iOS, Material on Android) and ships English and Russian localization.

## Tech stack

**Frontend/App:** Flutter, Dart (SDK ^3.10.8), Riverpod for state, go_router for navigation, `adaptive_platform_ui` for shell/chrome primitives, Flutter gen-l10n (en/ru).
**Audio:** `audio_service` (background media + lock-screen integration), `just_audio` (ambient/cue playback), `audio_session` (interruption and route-change events).
**Data/Infra:** Drift over SQLite (`drift_flutter`, `sqlite3_flutter_libs`) for structured storage; a separate JSON snapshot file for transient session recovery; `path_provider` for storage paths.
**Native:** Local `native_sheets` plugin (Dart + Swift/UIKit) for project-owned iOS selection sheets.
**Tooling/Tests:** `flutter_test`, `flutter_lints`, `build_runner` + `drift_dev` for codegen; GitHub Actions running root and package-local analyze/test.

## Architecture & engineering highlights

- **Three-layer session core with single responsibilities.** `SessionEngine` owns only business phase transitions (active interval → overlap → next → completed) and holds no notion of time. `SessionRuntime` owns wall-clock timing, pause/resume, and tick scheduling, and holds no business logic. `SessionController` orchestrates the two, maps state into audio commands, and persists snapshots. Keeping the engine time-free makes the transition rules trivially unit-testable.
- **Drift-aligned-tick scheduling instead of a periodic timer.** The runtime computes the delay to the next aligned one-second boundary from actual elapsed time rather than firing a naive periodic timer, so the displayed countdown does not drift over a long session, and recovers cleanly via a fallback recovery tick if scheduling throws.
- **Crash-safe recovery.** Active-session state is written to a JSON snapshot on lifecycle changes; on relaunch the controller replays the engine forward to the persisted interval/overlap/completed position and restores the runtime from either a started-at timestamp (running) or a paused elapsed value, reconstructing total session elapsed time across all prior intervals and overlaps.
- **Audio modeled as a command port.** `SessionController` talks to an `AudioCommandPort` interface implemented by `YogaAudioHandler` (a `BaseAudioHandler` bridging media-session controls). This decouples session logic from `audio_service` internals and lets the controller be tested without a real audio backend. Ambient, cue, and metronome concerns are split into dedicated services.
- **Interruption-aware playback.** The controller subscribes to `audio_session` interruption and becoming-noisy events, pausing on phone calls or headphone removal and resuming afterward only when the interruption type warrants it.
- **Extracted native plugin.** Reusable iOS UIKit selection sheets live in a standalone `native_sheets` package (Swift presenter/view-controller classes plus a Dart platform channel), keeping native modal behavior out of app-local widget composition and independently testable.

## Engineering challenges solved

- **Countdown drift and background suspension.** Naive periodic timers drift and stall when an app is backgrounded. Solution: derive remaining time from `now − startedAt` plus accumulated elapsed, schedule ticks aligned to second boundaries, and refresh from the timestamp on resume. Result: accurate time after suspension and across phase changes, proven by runtime and controller unit tests.
- **Cues clipped by phase changes.** Transition and completion bells were being cut off when the next phase started its audio bed. Solution: sequence cue playback so transition cues ring across phase boundaries and completion cues are not stopped by session-bed shutdown.
- **Snapshot write ordering.** Lifecycle events can trigger overlapping snapshot writes/clears. Solution: serialize snapshot mutations through a pending-future queue so writes and clears never interleave.
- **Overlap-phase restore math.** Restoring mid-overlap required reconstructing elapsed time differently from a normal interval boundary; a dedicated calculation folds prior interval and overlap durations to recover the correct session elapsed value.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-03-28 → 2026-05-06 |
| Commits | 52 (50 on main; 52 across branches) |
| Active development days | 10 |
| Lines of code | ~14,200 (hand-written, excl. deps & generated) |
| — App Dart (excl. generated) | 11,770 |
| — Test Dart (app) | 3,806 |
| — `native_sheets` Dart (excl. example) | 1,146 |
| — `native_sheets` Swift/UIKit | 924 |
| — Generated Dart (Drift `.g.dart` + l10n) | 4,465 (excluded above) |
| Languages | Dart (app + plugin), Swift (iOS plugin) |
| Localization | English, Russian (ARB) |
| Tests | 129 test functions across 22 files (125 app + 4 `native_sheets`) |
| Persistence | Drift/SQLite schema v2 (presets, preset_intervals, recent_launches) + JSON session snapshot |

## Screenshots

Available on request. (No in-repo screenshot assets; the repository contains audio assets under `assets/audio/` and platform launcher icons only.)
