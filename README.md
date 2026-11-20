# Job Application Assistant (Human-in-the-Loop)

A TypeScript and Playwright CLI pipeline designed to streamline remote job hunting through automated job aggregation, LLM-assisted compatibility scoring, and smart form autofill — **with mandatory human review before every submission**.

> **Design Philosophy:** Built as a productivity copilot, not an auto-apply spam bot. The script automates repetitive data entry and filtering while keeping final verification, custom screening questions, and the `Submit` action strictly under user control.

---

## Architecture & Workflow

The application operates as a decoupled 4-stage state machine (`PENDING_EVAL` ➔ `READY_TO_APPLY` / `REJECTED` ➔ `APPLIED`) persisted locally via a typed JSON store:

1. **Job Aggregation & Geo-Filtering (`src/scraper.ts`)**
   - Fetches remote job listings via public APIs/feeds (e.g., RemoteOK) matching configurable tech stack keywords.
   - Applies heuristic location filtering to exclude region-locked roles (e.g., US-state-specific restrictions).
   - Deduplicates listings by URL against the local store.

2. **AI Compatibility Scoring (`src/evaluator.ts`)**
   - Exports `PENDING_EVAL` roles to a structured evaluation batch (`data/pending_eval.json`).
   - Cross-references job descriptions against `data/profile.json` (tech stack, timezone overlap, and shift availability) to generate a `matchScore` (0–100), identify `redFlags`, and draft a tailored cover letter (`customPitch`).
   - Imports and validates approved roles (`matchScore >= 75`) into `READY_TO_APPLY`.

3. **Resilient Form Autofill (`src/autofill.ts`)**
   - Launches a persistent, non-headless Chromium instance (`~/.job-hunter-bot/chromium-profile`) to preserve active sessions across job boards.
   - Locates form inputs dynamically by inspecting `name`, `id`, `placeholder`, `aria-label`, and associated `<label>` elements.
   - Highlights populated fields in green, attaches the resume PDF, and fills cover letter textareas while safely skipping sensitive or ambiguous fields (e.g., salary expectations, current employer).

4. **Interactive CLI Orchestrator (`src/index.ts`)**
   - Provides a `readline/promises` terminal menu to trigger scans, export evaluation batches, launch autofill sessions for approved roles, and log `APPLIED` statuses.

---

## Tech Stack

- **Runtime & Language:** Node.js, TypeScript (Strict Mode)
- **Browser Automation:** Playwright (Chromium)
- **Execution:** `tsx`
- **Data Persistence:** Typed JSON Store (`src/store.ts`)

---

## Project Structure

```text
├── data/
│   ├── profile.example.json   # Template for candidate data, links, and constraints
│   ├── profile.json           # (Gitignored) Personal candidate profile
│   ├── jobs.json              # (Gitignored) Master state database
│   └── resume.pdf             # (Gitignored) Candidate CV
├── src/
│   ├── index.ts               # Interactive CLI menu
│   ├── scraper.ts             # Job fetcher & location/keyword filter
│   ├── evaluator.ts           # Batch exporter/importer for AI evaluation
│   ├── autofill.ts            # Playwright form detector & filler
│   └── store.ts               # Shared TypeScript interfaces & JSON persistence
├── .gitignore
├── package.json
└── tsconfig.json
