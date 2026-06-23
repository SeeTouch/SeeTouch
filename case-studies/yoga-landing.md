# Multilingual Yoga Landing

> Multilingual marketing site for a yoga teacher, edited entirely through a CMS. *(Private client — shared without brand name.)*

<kbd>Role: Solo developer</kbd> · <kbd>Astro + React + Tailwind v4</kbd> · <kbd>2026-03-29 → 2026-05-11</kbd> · <kbd>Status: Production</kbd>

## Overview

A polished, server-rendered marketing site for a Kundalini yoga teacher, built to showcase a portfolio of past retreats, surface upcoming events, and convert visitors into clients via direct messaging channels. The site is fully trilingual (Russian, English, German) and is entirely content-managed: every headline, photo gallery, event, and review is editable by a non-technical owner through a cloud CMS, with no developer involvement after launch. It ships on Cloudflare's edge runtime.

## What it does

- Presents a single-page landing (hero, about, services, event preview, contacts) plus standalone event and services pages
- Serves three languages from one codebase via locale-prefixed routes, with the default locale at the root
- Renders individual shareable event pages with photo galleries, reviews, and rich CMS-authored descriptions
- Lets the owner manage all content (homepage, events, services) through Keystatic Cloud — including image uploads
- Computes event status (upcoming vs. past) automatically from dates rather than trusting a manual flag
- Emits a multilingual sitemap with hreflang alternates and full Open Graph / Twitter / JSON-LD metadata for SEO

## Tech stack

**Frontend/App:** Astro 5 (SSR, `output: server`), React 19 islands, Tailwind CSS v4 (Vite plugin), Playfair Display + Inter via Fontsource, `yet-another-react-lightbox` for galleries
**Backend/Services:** Astro API routes on Cloudflare Workers; Cloudflare R2 for user-uploaded images
**Data/Infra:** Keystatic Cloud CMS (singletons + collections, YAML + Markdoc); Cloudflare Pages deploy via Wrangler
**Tooling/Tests:** TypeScript, Astro check; no automated test suite

## Architecture & engineering highlights

- **Component system follows atomic design** — `atoms/`, `molecules/`, `organisms/` — keeping the Astro components composable and the page files thin orchestrators.
- **Manual i18n layer** built on `[locale]` dynamic routes plus a small `i18n/utils.ts` (`t`, `localized`, `localePath`, `getAlternateUrls`). The default locale (RU) lives at the root path while EN/DE are prefixed, with graceful per-key fallback to the default language. CMS content uses a reusable `localeText` / `localeDocument` field factory so every field is authored in all three languages.
- **CMS modeling in `keystatic.config.ts`** defines a `homepage` singleton and `events` / `services` collections, a custom `photoGroup` document component block (rendered by a React island), and image fields wired to public paths — letting editors lay out galleries inside rich text.
- **Data access is isolated** in `lib/events.ts`, `lib/homepage.ts`, and `lib/services.ts` using the Keystatic reader, so pages stay declarative. Every CMS read is wrapped in try/catch so the build degrades gracefully when the CMS is not yet configured.
- **SEO is first-class:** canonical URLs, hreflang alternates (including `x-default`), per-locale Open Graph locale tags, JSON-LD Organization schema, and a dynamically generated `sitemap.xml` that enumerates every event slug across all locales.

## Engineering challenges solved

- **Running Astro SSR + React on Cloudflare Workers.** The Workers runtime lacks `MessageChannel`, which React's server renderer depends on. Solved with a custom Vite `renderChunk` plugin that injects a `MessageChannel` polyfill into the generated `_worker.js` and renderer chunks, plus production-only SSR resolve conditions (`workerd`/`worker`/`browser`) and a `react-dom/server` → `react-dom/server.browser` alias. This was the difference between the site building and crashing on the edge.
- **Secure image uploads at the edge.** The `/api/upload` route validates the request Origin against an allowlist, enforces a 10 MB cap, and verifies file type by inspecting magic bytes (JPEG/PNG/WebP signatures, with an extra RIFF/"WEBP" check) rather than trusting the client-supplied MIME type, then stores objects in R2 under a random UUID key. This blocks content-type spoofing and arbitrary uploads.
- **Trustworthy event status.** Rather than relying on an editor-set "upcoming/past" flag that inevitably goes stale, status is derived at render time from the event's end-of-day date, so the timeline is always correct without manual maintenance.
- **Defense-in-depth headers** shipped via Cloudflare `_headers`: `X-Frame-Options: DENY` (relaxed to `SAMEORIGIN` only for the CMS admin path), `nosniff`, a restrictive `Permissions-Policy`, and one-year immutable caching for hashed static assets.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-03-29 → 2026-05-11 |
| Commits | 94 |
| Active development days | 12 |
| Lines of code | ~4,486 (excl. deps) |
| Languages | Astro 67%, TS 18%, CSS 11%, JSON 3%, JS/TSX 2% |
| Tests | — (no automated suite) |

## Screenshots

Not shared publicly. Available privately on request.
