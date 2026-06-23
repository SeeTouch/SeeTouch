# DNA Mandala

> Real-time 3D system mapping a birth chart to sacred geometry and generative sound.

<kbd>Role: Solo developer</kbd> · <kbd>React Three Fiber / Three.js</kbd> · <kbd>2026-04-05 → 2026-04-08</kbd> · <kbd>Status: Prototype</kbd>

## Overview

DNA Mandala is an interactive WebGL application that unifies six symbolic systems — Human Design, the I Ching, DNA codons, sacred geometry, numerology and generative music — around their shared structural constant: the number 64. A user enters a birth date and time; the app computes a Human Design chart from real planetary ephemeris, projects the resulting 64 activations into an animated 3D mandala, and drives a deterministic generative soundtrack from the same data. It is a math-heavy exploration of how a single seed (a moment in time) can be expressed simultaneously as geometry, color and sound.

## What it does

- Computes a Human Design bodygraph from a birth date/time: 13 planetary positions for both the Personality (birth) and Design (~88 days prior) charts, each mapped to gate, line, color and tone.
- Renders an animated 3D cosmic mandala of 64 gates with orbiting pulsars, particle trails, bloom post-processing and a deep multi-layer starfield.
- Offers seven view modes for the same data — mandala, helix, torus, sphere, Lorenz and Rössler chaotic attractors, and Ba Gua — plus live sliders for flow, chaos, inertia, trail length, density and bloom.
- Generates a deterministic multi-track soundtrack (drone, melody, bass, arpeggio, harmony) tuned to 432 Hz, where the same birth date always produces the same music.
- Exports the generated composition as a standard MIDI file for use in any DAW.

## Tech stack

**Frontend/App:** React 18, TypeScript, Vite, Tailwind CSS v4
**3D:** Three.js, React Three Fiber, Drei, @react-three/postprocessing (Bloom, Vignette)
**Audio:** Tone.js (synthesis, transport, mixer bus routing), midi-writer-js (MIDI export)
**Astronomy/Math:** astronomy-engine (planetary ephemeris), swisseph-wasm, simplex-noise
**State:** Zustand (five focused stores)
**Tooling/Tests:** Vitest, ESLint, Prettier, Husky + lint-staged, deployed via Vercel

## Architecture & engineering highlights

- **Strict separation of pure computation from rendering.** The `engine/` layer (numerology, ephemeris-to-gate mapping, sacred-geometry math) and the entire `music/` data layer are pure functions with no UI or audio dependencies, which is what makes them unit-testable in isolation. Audio side effects live only in the Tone.js mixer, synth factory and conductor.
- **Real ephemeris-based Human Design calculation.** `hd-calculator.ts` derives ecliptic longitudes for Sun, Moon, the lunar nodes and seven geocentric planets, then maps each to a precise gate/line/color/tone via the 64-gate wheel (5.625° per gate, subdivided down to 1/36 of a degree). The Design chart is found with a true solar-arc search (`SearchSunLongitude` for the point 88° of solar travel before birth) rather than a fixed day offset.
- **LOD rendering pipeline.** A distance-driven `LODController` swaps between three representations — 64 gate points at far range, 384 GPU-instanced spheres at mid range, and a full point cloud plus per-activation pulsars up close — with opacity cross-fading so transitions are seamless. The 384 line markers are drawn with a single `InstancedMesh` and a per-instance color attribute.
- **Deterministic generative music.** A birth date is hashed into a 32-bit seed driving a Mulberry32 PRNG; a `Conductor` runs one Tone.Transport loop that ticks five tracks in sync. Each track routes through a four-bus mixer (drone / melody / ambient / binaural) into a master compressor + limiter chain. Synth voices are chosen per amino-acid class of the active codon and detuned −31.77 cents to land on 432 Hz tuning.
- **Symbolic data model.** The I Ching, DNA and Human Design layers are unified through JSON reference tables and a `transform.ts` module implementing the Harriman nucleotide-to-binary encoding, Fu Xi ↔ King Wen mappings, trigram lookup and codon-ring grouping.

## Engineering challenges solved

- **Eliminating visual jumps when users drag sliders.** Naively recomputing pulsar positions from changed parameters caused jarring teleports and trail artifacts. The pulsar integrates an accumulated angle (only the per-frame delta is added) and applies adaptive lerp smoothing — fast for normal motion, deliberately slow for large jumps — so parameter changes are absorbed without breaking the continuous trails.
- **Keeping motion trails stable across mode switches.** Trail length is controlled through the attenuation cutoff of a fixed-size buffer rather than by remounting the trail, avoiding flicker when switching among the seven view modes.
- **Mapping continuous orbital geometry to discrete symbols.** Converting a continuous ecliptic longitude into a discrete gate/line/color/tone required careful 360°-wraparound handling and clamping at boundaries to keep every activation valid.
- **Pluggable view geometries.** A `STRATEGIES` table lets each view mode (including Lorenz and Rössler attractors) supply its own position function while sharing the same chaos, flow and smoothing pipeline.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-04-05 → 2026-04-08 |
| Commits | 30 |
| Active development days | 3 |
| Lines of code | ~6,970 TS/TSX (excl. deps); ~1,080 JSON data |
| Languages | TypeScript ~45% (3,137 LOC), TSX ~33% (2,335 LOC), JSON data ~15%, CSS <1% |
| Tests | 161 cases across 11 Vitest suites (~1,490 LOC of tests) |

## Screenshots

Reference assets in the repository:

- `docs/references/dna_mandala_codons.svg`
- `docs/references/codon-binary-notes.png`
- `docs/references/hd-gate-sequence.png`
- `docs/references/trigram-notes-mapping.png`

Live application screenshots available on request.
