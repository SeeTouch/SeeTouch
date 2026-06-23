# SeeTouch

> Open-core, edge-first smart-home automation platform for pro AV/KNX/Modbus integrators.

<kbd>Role: Solo developer</kbd> · <kbd>TypeScript · Node 22 · pnpm/Turbo monorepo</kbd> · <kbd>2026-04-28 → 2026-05-07</kbd> · <kbd>Status: MVP (Phase 1 backend on main)</kbd>

## Overview

SeeTouch is a smart-home automation platform built for professional AV and building-control integrators — the segment between DIY (Home Assistant) and proprietary systems (Crestron, AMX, Lutron). Each installed site runs a local edge box (Raspberry Pi 5 / mini-PC / Mac mini) that orchestrates TVs, AV gear, lighting, climate, and security over TCP, IR, RS-232, KNX, and Modbus. The cloud is optional: the edge core runs standalone, and project configuration is versioned in git so a smart home gets CI/CD-grade change control. The result is an auto-generated client UI for homeowners and a visual configurator (Studio) for installers, sharing one type-safe API.

## What it does

- Unifies multimedia, lighting, climate, security, and pro-AV subsystems under a single auto-generated end-user app.
- Runs device drivers in isolated Worker Threads so a misbehaving or crashing driver cannot take down the hub.
- Bridges protocols: e.g. a KNX group-address change can drive a Modbus coil write, with value transforms and loop-guarding against feedback storms.
- Discovers devices on the network via mDNS/SSDP/SNMP scanning.
- Streams live device state to clients over WebSockets, scoped per paired device and per project.
- Lets integrators pair devices, build rooms/zones, and lay out controls as data through a tRPC API surface.

## Tech stack

**Frontend/App:** React 19, Vite, TailwindCSS 4, Radix UI, Zustand, React Router; two apps — client-web (homeowner) and studio-web (integrator).
**Backend/Services:** Node.js 22 + TypeScript, Fastify 5, tRPC v11, uWebSockets.js for real-time pub/sub, Pulse internal event bus (mitt-based).
**Data/Infra:** SQLite + Drizzle ORM with 26 versioned migrations; Argon2 (`@node-rs/argon2`) for credential hashing; Docker for the local dev/bench loop.
**Tooling/Tests:** pnpm workspaces + Turborepo, Vitest, ESLint + Prettier, strict TypeScript (no `any`), Conventional Commits.

## Architecture & engineering highlights

- **Monorepo, 19 packages + 3 apps.** Clean subsystem boundaries: `core` (orchestration), `pulse` (event bus), `memory` (SQLite/Drizzle), `drivers-runtime` (Worker Thread pool + host + gateway), `driver-sdk`/`driver-schema` (manifest-driven driver contract), `api` (tRPC ~20 routers), `bridge-runtime` (protocol bridging), `scanner`, `project-loader`, `cli`, plus four real example drivers (Sony Bravia, Samsung TV, NVIDIA Shield, Zigbee curtains).
- **Driver isolation via Worker Threads.** A `WorkerPool` spawns each driver into its own worker with a typed message protocol, per-driver log and state-history ring buffers (500 log / 200 state entries) for a Dev Console, and idempotent spawn checks. Validated in a dedicated Phase 0 PoC before any product code was written.
- **Project-scoped real-time routing.** The uWebSockets.js server enforces a deliberate dispatch contract: bootstrap actors receive every event, while paired auth-device sockets only receive events carrying a matching `projectId`. Hub-wide system events are intentionally withheld from device sockets — a security boundary expressed directly in the event router.
- **Protocol bridge with loop-guard.** `bridge-runtime` compiles bridge definitions, evaluates value transforms with a sandboxed expression evaluator, and suppresses the back-emit on the same bridge so a KNX↔Modbus round-trip cannot oscillate. Covered by an end-to-end test that runs against the real Pulse bus, not mocks.

## Engineering challenges solved

- **Mocks that lie about runtime contracts.** A multi-reviewer audit found that 499 passing unit tests still hid a critical bug: workers would silently exit after module load because Tier-1 driver packages exported library factories, not the Worker-protocol entry the pool expected — invisible to mocked tests. The response was a standing discipline: phase-closure audits must include a runtime-contract trace and at least one integration test against the real Pulse/WorkerPool/DB. Three failure modes — silent-exit, data loss, and resource leaks — are defended from day one; everything else is deliberately deferred with inline `TODO(production-rigor)` markers.
- **Credential hygiene at the edges.** Token verification uses constant-time comparison (`timingSafeEqual`), and driver log output is run through a sanitizer that redacts PSK/password/token/secret patterns before broadcast, so credentials never leak into the Dev Console stream.
- **Dual-mode worker entry resolution.** The worker host is resolved as `.ts` under `tsx` in dev and `.js` from `dist/` in production from a single code path, removing a class of "works in dev, breaks in build" failures.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-04-28 → 2026-05-07 |
| Commits | 278 |
| Active development days | 10 |
| Lines of code | ~81,000 TypeScript/TSX (excl. deps, build, tests); ~32,000 additional in tests |
| Languages | TypeScript/TSX ~94%, YAML ~10K, plus JSON/Markdown config & docs |
| Tests | 1,058 cases across 100 Vitest files |

## Screenshots

UI design source assets are present in the repository (`docs/design/sketch-source/*.png` — 11 mockups incl. room views, AV remotes, splash and storyboard) and a logo (`apps/studio-web/src/icons/seetouch-logo.svg`). Application runtime screenshots: available on request.
