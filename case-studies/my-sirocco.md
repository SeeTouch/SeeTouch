# M/Y Sirocco — AV & Automation Systems Audit

> A structured as-built audit framework documenting the AV, control, network, and CCTV systems of a 47 m superyacht.

<kbd>Role: AV/automation consultant (audit lead)</kbd> · <kbd>Domain: marine AV/control systems</kbd> · <kbd>May 2026</kbd> · <kbd>Status: Documentation deliverable (pre-visit scaffold)</kbd>

## Overview

M/Y Sirocco is a professional audio-video and automation systems audit for a 47-metre Heesen superyacht (2006 build, 2013 refit). The objective is to document every AV, control, network, and CCTV system aboard "as-built," surface defects and risks, and give the owner and management company a defensible basis for a modernization decision. This case study covers the structured documentation system and methodology that drive the engagement: a repeatable, two-dimensional (zone × system) framework designed for a fast on-site survey followed by office post-processing into a client-ready report.

The deliverable demonstrates domain expertise in marine AV/control integration — Crestron, Savant, AMX, KNX, Lutron and similar platforms — and the discipline of turning a chaotic field survey into auditable, versioned documentation. It complements automation-platform work by encoding how a real superyacht's control, audio, and video subsystems are inventoried, diagrammed, and assessed.

## What it does

- Defines a complete audit scope across guest zones, technical spaces, and equipment racks, with explicit in-scope / boundary / out-of-scope lines
- Provides a repeatable per-zone template (inventory, findings, signal flow, recommendations, rack/cabling diagrams, photo log) instantiated for 25 zones across five decks
- Captures cross-zonal systems (audio matrix, video/HDMI routing, control topology, network/VLANs, CCTV, satellite TV, SATCOM, intercom) as a second navigation axis
- Documents equipment racks as first-class objects: 1U elevation, inventory, power distribution, and cable mapping
- Records architecture decisions (ADRs) and open questions (RFI log) with a punch-list severity scheme (Critical/High/Medium/Low)
- Produces phased client deliverables: an audit proposal, an as-built report, an aggregated equipment inventory/BOQ, system diagrams, and a high-level cost estimate
- Automates report assembly (markdown → PDF via a manifest-driven pipeline) and inventory aggregation from per-zone CSVs

## Systems & domains covered

**AV systems:** Audio (DSP, matrix/zones, amplification, speakers, sources), Video (HDMI/IP matrix, displays, sources incl. SAT/Apple TV/VOD/Kaleidescape), dedicated cinema room.
**Control & automation:** Multi-platform control system assessment (Crestron / Savant / AMX / Control4 / RTI / KNX / Lutron HomeWorks), iPad and wall-panel GUIs, scene programming, source-code/ownership tracking, integration points.
**Network & IT:** LAN/VLAN topology, Wi-Fi coverage (AV-relevant), VSAT / 4G/5G / Starlink WAN, IP planning.
**Other subsystems:** CCTV (cameras, NVR, retention), intercom/paging, satellite TV, plus integration touchpoints for lighting, shades/blinds, and HVAC.
**Method & tooling:** Markdown knowledge base (Obsidian vault with Dataview dashboards), draw.io for rack elevations / cabling / signal-flow diagrams, Mermaid for inline flows, Pandoc + XeLaTeX for PDF compilation, shell scripts for scaffolding and inventory aggregation, Git/GitHub workflow with issue templates (RFI / audit-finding / decision-needed) and ADRs.

## Architecture & engineering highlights

- **Two-dimensional information model.** A deliberate zone-first / system-second structure (captured in an ADR) mirrors the physical walk-through on board while still allowing a full cross-zonal view of any single system — the "where" and the "what" are both first-class.
- **Field-to-report workflow.** The framework is built around dictation on board (new-zone scaffold → inventory → findings → photos) feeding a structured post-visit pipeline (diagram redraw, inventory aggregation, report compilation), so the on-site time is spent capturing, not formatting.
- **Repeatable templates.** A zone template, ADR template, RFI/finding templates, and inventory CSV schemas make the methodology reusable across future vessels, not a one-off.
- **Build automation.** Manifest-driven `build-report.sh` (Pandoc + XeLaTeX, TOC, numbered sections) and `aggregate-inventory.sh` turn scattered per-zone notes into a single client PDF and a consolidated bill of quantities.
- **Bilingual discipline.** Working notes, ADRs, and brainstorms are kept in the working language; all client-facing deliverables and reference documentation are authored in English, with diagram labels standardized in English.

## Engineering challenges solved

- **Turning an unstructured field survey into auditable output.** The two-axis model plus per-zone templates and severity-tagged findings convert a fast, voice-driven walk-through into consistent, comparable, versioned documentation.
- **Avoiding inventory duplication.** Equipment is captured once per zone (the source of truth) and aggregated automatically into system-level and project-level inventories, resolving the classic zone-vs-system double-entry problem.
- **Privacy by design.** The structure separates public-sourced vessel facts from sensitive client data; stakeholder PII, IP addressing, and credentials are deliberately left as placeholders to be filled on board, never committed to the repository.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-05-17 → 2026-05-18 |
| Commits | 10 |
| Active development days | 2 |
| Project type | Documentation / consulting deliverable (no application code) |
| Markdown documents | 114 substantive docs |
| Document sections (headings) | 700+ |
| Decks documented | 5 (flybridge, wheelhouse, main, lower, tech-spaces) |
| Zones scaffolded | 25 |
| Cross-zonal systems covered | 11 (audio, video, control, network, satellite TV, SATCOM, CCTV, intercom, lighting, shades/blinds, HVAC integration) |
| Equipment racks | 3 (main AV rack, bridge rack, distribution points) |
| Diagrams (draw.io) | 5 (audio, video, network, 2 rack elevations) |
| Inventory schemas (CSV) | 4 aggregatable inventories + templates |
| Automation scripts | 4 (scaffold, inventory aggregation, report/PDF build) |

## Diagrams & assets

- `assets/ga-plans/01-flybridge-deck.png` … `04-lower-deck.png` — general-arrangement deck plans
- `03-systems/{audio,video,network}/*.drawio` — system signal-flow and topology diagrams
- `04-racks/{main-av-rack,bridge-rack}/elevation.drawio` — rack elevations
- Per-zone cabling/rack diagrams and photo logs — populated on site; **available on request**
