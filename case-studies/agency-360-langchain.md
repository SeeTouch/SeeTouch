# Agency 360 — Multi-Agent Onboarding System

> Code-first LangGraph backend that automates hospitality recruitment onboarding end to end.

<kbd>Role: Solo developer</kbd> · <kbd>Python · LangGraph · FastAPI</kbd> · <kbd>Jan–Mar 2026</kbd> · <kbd>Status: Production</kbd>

## Overview

Agency 360 automates the full candidate onboarding cycle for a hospitality recruitment agency. It replaces a brittle low-code (n8n) workflow with a code-first, asynchronous multi-agent system built on LangGraph. A conversational agent ("Lisa") runs candidates through a deterministic onboarding sequence over chat, while specialized agents parse CVs, analyze photos, and detect work-history anomalies. The design goal was reliability: a deterministic state machine drives the process, and the LLMs act as a "smart interface" for extraction and natural language, never as the controller of the flow. Development ran continuously over roughly two and a half months, growing from a working prototype into a full system with a cross-job work-history analyzer, a references and summary builder, a data-correction layer, and context-aware step skipping.

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
| Timeline | 2026-01-08 → 2026-03-17 |
| Commits | 197 (single repository, all authored by me) |
| Active development days | 24 |
| Lines of code | ~12,100 Python (54 authored modules, excl. venv/site-packages) |
| Architecture | 4 specialized agents · 3 compiled LangGraph pipelines · provider-agnostic LLM factory (Anthropic + Gemini + OpenAI) |
| Tests | LLM-driven end-to-end simulator with persona scenarios |

> The project was developed in a single Git repository using two working trees — a stable snapshot branch and an active development branch where the bulk of the 197 commits live. A detailed `CHANGELOG.md` and `ROADMAP.md` track the day-by-day iteration on the conversational logic, extraction agents, and onboarding state machine.

## Screenshots

No UI screenshots (backend service). Architecture and agent documentation, a detailed `CHANGELOG.md`, and a `ROADMAP.md` document the system and its development history. A live conversational demo (the "Lisa" onboarding flow over chat) is available on request.
