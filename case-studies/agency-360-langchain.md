# Agency 360 — Multi-Agent Onboarding System

> Code-first LangGraph backend that automates hospitality recruitment onboarding end to end.

<kbd>Role: Solo developer</kbd> · <kbd>Python · LangChain / LangGraph</kbd> · <kbd>Jan–Mar 2026</kbd> · <kbd>Status: MVP</kbd>

## Overview

Agency 360 automates the full candidate onboarding cycle for a hospitality recruitment agency. It replaces a brittle low-code (n8n) workflow with a code-first, asynchronous multi-agent system built on LangGraph. A conversational agent ("Lisa") runs candidates through a deterministic onboarding sequence over chat, while specialized agents parse CVs, analyze photos, and detect work-history anomalies. The design goal was reliability: a deterministic state machine drives the process, and the LLMs act as a "smart interface" for extraction and natural language, never as the controller of the flow.

## What it does

- Conducts a guided, stateful onboarding conversation across 20 defined steps (name, position, location, employment status, notice period, nationality, DOB, contacts, languages, photos, references, and more).
- Parses uploaded CVs (PDF/DOCX) with a vision model, extracting hospitality-specific data: employment types, normalized dates and job titles, education, languages, and references.
- Analyzes candidate photos for professional suitability and demographic consistency, flagging gender/photo mismatches against the profile.
- Detects work-history anomalies — overlapping employment, gaps over three months, short tenure, current-gap, and status mismatches — and generates targeted clarification questions on the fly.
- Persists every session in PostgreSQL so interrupted dialogues resume with full context, and syncs each confirmed field to an external CRM event by event.
- Produces a structured candidate summary, including an AI behavioral assessment, on completion.

## Tech stack

**App/Gateway:** FastAPI (async HTTP gateway: `/api/message`, `/api/cv/parse`, `/api/photo/analyze`), Uvicorn
**Orchestration:** LangGraph (three compiled `StateGraph` pipelines), LangChain
**LLMs:** Provider-agnostic factory over OpenAI (GPT-4o / 4o-mini, including vision), Anthropic Claude, and Google Gemini; Perplexity for enrichment/fact-checking
**Data/Infra:** PostgreSQL via SQLAlchemy 2.0 async + asyncpg, Pydantic v2 schemas, Docker / Docker Compose
**Tooling/Tests:** LLM-driven end-to-end simulator with persona scenarios; pypdf / PyMuPDF for document handling

## Architecture & engineering highlights

- **Deterministic orchestration, AI as interface.** The onboarding flow lives in a code-defined sequence (`onboarding_sequence.py`) with explicit ordering, skip/ask conditions, and dependency-based cascading resets. The LLM extracts data and phrases responses but cannot deviate from the business process, which structurally eliminates off-script hallucination.
- **Three LangGraph pipelines.** Lisa's conversation graph (triage → load state → process input → determine step → generate response), a 17-node CV-parsing pipeline (download → parse → process → map → enrich → photo-analysis → multi-entity CRM sync), and a photo-analysis graph — each a compiled `StateGraph` over typed state.
- **Structured Outputs everywhere.** Extraction, triage classification, and photo analysis use `with_structured_output` against Pydantic models, so LLM responses parse deterministically and conform to database contracts rather than relying on free-text parsing.
- **Provider-agnostic LLM factory.** A single `get_llm(context=...)` resolves provider, model, temperature, timeout, and retry policy per role (chat vs. enrichment vs. CV parsing) from environment config, enabling hot-swapping models without code changes.
- **Stateful, resumable sessions.** Session state (completed/asked steps, per-step attempt counters, discrepancy flags, dynamic question queues) is persisted to PostgreSQL on every transition, so long-running or interrupted conversations recover cleanly.

## Engineering challenges solved

- **Loop and refusal control.** Free-form chat tends to trap LLM agents in clarification loops. The system tracks per-step refusal and clarification counts with configurable caps, a part-based refusal strategy (intro vs. data steps), and a loop-breaker that force-advances after repeated failed clarifications.
- **Deterministic date-of-birth handling.** DOB inputs are notoriously ambiguous (`6.8` vs `8.6`, partial dates, relative phrases). A code-level validator overrides LLM guesses: it detects ambiguous numeric formats, expands partial replies using the year from a prior disambiguation question, and normalizes to ISO before any CRM write.
- **Cross-source discrepancy detection.** Names from chat vs. CV, and gender from photo vs. profile, are reconciled deterministically with dedicated one-shot clarification steps, avoiding both silent data corruption and repeated nagging.
- **Strict language gating.** A triage node classifies intent and language; non-English messages are blocked from being processed as onboarding data, with deterministic guards so valid short factual answers (e.g. a city name) are not misclassified.

## By the numbers

| Metric | Value |
|---|---|
| Timeline | 2026-01-08 → 2026-03-05 (first → last commit) |
| Commits | 23 |
| Active development days | 11 |
| Lines of code | ~10,800 Python (authored, excl. venv/site-packages) |
| Languages | Python (64 authored modules) + 11 Markdown prompt files; YAML/JSON config |
| Tests | LLM-driven E2E simulator (runner + 6 persona scenarios); no unit-test suite |

> Note: the working tree contains ~5,000 `.py` files, but all but ~64 are vendored dependencies inside the virtualenv and are excluded from the LOC figure above. Git history in the snapshot reflects 23 commits over 11 active days; metrics here are taken directly from the repository.

## Screenshots

No UI screenshots (backend service). Real E2E test assets exist under `tests/e2e/assets/` (anonymized photos and sample CVs). Architecture and agent documentation is available under `backend/docs/` (`ARCHITECTURE.md`, `AGENTS.md`). Demonstration available on request.
