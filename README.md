# AI Research Assistant Platform

An n8n-based automation platform that helps researchers, students, and faculty stay on top of new academic publications — without having to manually check arXiv every day. Register a topic once, and the system searches for new papers, summarizes them with AI, files them into a searchable repository, and emails you a weekly digest.

Built as a capstone project for Summer School '26 (n8n track).

---

## Why this exists

Keeping up with new research in a fast-moving field usually means checking arXiv or Google Scholar every few days, skimming a pile of abstracts, and hoping you didn't miss anything important while you were busy with something else. This project automates that entire loop: register your interests once, and let the pipeline handle discovery, summarization, and organization on its own.

---

## How it works

The platform is built as **5 independent n8n workflows** that share a single Google Sheet as their data store, rather than being tightly wired together. Each workflow reads what it needs, does its job, and writes results back — tagging rows with a `status` (`new` → `summarized` → `archived`) so the next workflow knows what's ready.

```
[Google Form]
     │
     ▼
1. Research Topic Registration  ──────►  topics sheet
                                              │
                                              ▼
2. Paper Search & Collection (daily)  ──►  papers sheet (status: new)
                                              │
                                              ▼
3. AI Summarization & Keyword Extraction  ─►  papers sheet (status: summarized)
   (hourly)                                   │
                                              ▼
4. Reference Management & Repository  ────►  papers sheet (status: archived)
   (hourly)                                   +  Google Doc repository
                                              │
                                              ▼
5. Weekly Digest & Notifications  ─────────►  Gmail → user inbox
   (weekly)
```

> 📎 *Insert the full architecture diagram image here.*

---

## Workflows

| # | Workflow | Trigger | What it does |
|---|---|---|---|
| 1 | Research Topic Registration | Google Sheets Trigger (form-linked) | Captures a user's research interests from a Google Form, validates the submission, stores it, and sends a confirmation email. |
| 2 | Paper Search & Collection | Cron — daily | Searches arXiv for each registered topic, parses the results, deduplicates, and stores new papers. |
| 3 | AI Summarization & Keyword Extraction | Cron — hourly | Summarizes each new paper's abstract using Groq (Llama 3.3 70B) and extracts keywords. |
| 4 | Reference Management & Repository | Cron — hourly | Categorizes each summarized paper with AI and files it into a running Google Doc repository. |
| 5 | Weekly Digest & Notifications | Cron — weekly | Matches archived papers to each user's topic and emails a personalized digest with an AI-written intro. |

Exported workflow JSON files for all 5 are in [`/workflows`](./workflows).

---

## Tech stack

- **[n8n](https://n8n.io)** — self-hosted, workflow orchestration
- **[Groq](https://groq.com)** (`llama-3.3-70b-versatile`) — summarization, keyword extraction, categorization, digest intro generation
- **[arXiv API](https://arxiv.org/help/api)** — paper discovery (free, no key required)
- **Google Sheets** — shared data store across all 5 workflows (`topics`, `papers`, `logs` tabs)
- **Google Docs** — running paper repository
- **Google Forms** — user registration front-end
- **Gmail** — confirmation emails + weekly digests

Everything here runs on free tiers — no paid APIs or infrastructure required.

---

## Setup

1. **Create the Google Sheet**
   Create a spreadsheet called `Research Assistant DB` with three tabs: `topics`, `papers`, `logs`. Column headers are listed in [`docs/sheet-schema.md`](./docs/sheet-schema.md).

2. **Create the Google Form**
   A short form collecting name, email, and research topic/keywords, linked to a response spreadsheet.

3. **Create the repository Google Doc**
   A blank Google Doc titled `Research Repository` — Workflow 4 appends to this.

4. **Get a Groq API key**
   Free at [console.groq.com](https://console.groq.com).

5. **Set up Google OAuth credentials** (Sheets, Drive, Docs, Gmail APIs)
   Required if self-hosting n8n — see [`docs/google-oauth-setup.md`](./docs/google-oauth-setup.md) for the full walkthrough.

6. **Import the workflows**
   Import each JSON file from `/workflows` into your n8n instance, connect your credentials on each node, and activate all 5.

---

## Known limitations

- **Semantic Scholar** was originally planned as a second paper source alongside arXiv, but was dropped from this version due to persistent rate-limiting on the public endpoint and an API key application that wouldn't validate an institutional email address. arXiv alone currently powers paper discovery. Re-adding Semantic Scholar later just needs a Wait node + retry settings on the existing HTTP Request branch.
- `topic` and `keywords` currently share one form field — splitting these into a broad category plus specific search keywords would allow more precise digest matching.
- The repository is a single Google Doc rather than a proper searchable index — fine at this scale, but would need a real search layer if the paper volume grew significantly.
- No human-approval step currently sits before a paper is archived or a digest is sent — the pipeline is fully automated end to end.

---

## Documentation

Full problem analysis, architecture rationale, and per-workflow breakdowns (including the exact AI prompts used) are in [`docs/Problem-Analysis-and-Workflow-Documentation.docx`](./docs/Problem-Analysis-and-Workflow-Documentation.docx).

---

## Demo

> 📎 *Insert demo video link here once recorded.*

---

## Author

Anshul Singh — Summer School '26, n8n Capstone Project
