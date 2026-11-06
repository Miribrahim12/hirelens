# HireLens

Azerbaijani HR screen that ranks PDF CVs against one job. A jury can run the full story with **Nümunə ilə sına** and no API call.

## Problem

Reading every CV by hand is slow, uneven, and easy to bias with name, age, or photo. A live demo also fails if the model quota is gone.

## Solution

HireLens sends each PDF to Gemini as inline data and shows a score, verdict, evidence, and interview questions. The final score is a weighted sum computed in the app, not by the model. Must-have gaps cannot be a strong match. The sample button loads a Junior Data Analyst role and three fictional candidates instantly. Uploaded CVs are not stored.

## Architecture

```
Browser (Next.js)
  job text + PDF base64
        |
        v
POST /api/analyze
  validate PDF (max 4 MB)
  Gemini inline PDF + JSON schema
  retries on 429 / 503 / UNAVAILABLE
  optional GEMINI_FALLBACK_MODEL once
        |
        v
Weighted score, must-have verdict cap, evidence check
        |
        v
Ranking table, side panel, compare, CSV, invitation text
```

There is no database and no login. Files live only in the browser tab until refresh or reset. The API route does not write them to disk.

The sample path never calls `/api/analyze`. Its JSON is in `src/lib/demo-screening.ts` and includes a fictional suspicious CV. Rules live in the model system instruction. The CV and job text are treated as data. If instruction-like text is detected, the app caps the score at 40 and leaves the hiring decision to HR.

At most three analyses run at once. Each row updates as soon as that CV finishes: Oxunur, Analiz olunur, Hazırdır.

## Setup

```bash
npm install
cp .env.example .env.local
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). For a live demo, click **Nümunə ilə sına**. A real Gemini key is only needed when you upload PDFs.

List models for the key in `.env.local`:

```bash
node scripts/list-models.mjs
```

## Environment variables

| Name | Required | Purpose |
| --- | --- | --- |
| `GEMINI_API_KEY` | For real PDFs | Server-only key. Never commit it. |
| `GEMINI_MODEL` | For real PDFs | Model name. Read only from this variable. The example is `gemini-3.5-flash`. |
| `GEMINI_FALLBACK_MODEL` | No | Tried once if the primary model stays unavailable. Empty by default. |

Copy `.env.example` to `.env.local`. The example file has an empty key.

## Scripts

- `npm run dev` — local development
- `npm run build` — production build
- `npm run start` — serve the production build
- `npm run lint` — ESLint

## Roadmap

- Save a shortlist only when an HR team asks for an account
- Calibrate the rubric on a labeled set of CVs
- Export a one-page interview pack as PDF
- Let the candidate download the rewritten summary
