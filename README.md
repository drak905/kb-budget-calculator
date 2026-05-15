# Knowledge Core — Budget Calculator v2

**Live OpenRouter Pricing · LLM + RAG Cost Estimator for Enterprise Deployments**

A standalone, single-file web application that estimates the monthly operational cost (OPEX) and one-time setup cost (CAPEX) of running a Knowledge Core RAG (Retrieval-Augmented Generation) system. Built for **PNJ organizational deployment planning**, with live model pricing fetched from [OpenRouter](https://openrouter.ai).

- **Live at**: `https://drak905.github.io/kb-budget-calculator/`
- **Hosting**: GitHub Pages (free, auto-deploys on push)
- **No backend** — everything runs in the browser as client-side JavaScript

---

## What This Calculator Estimates

A Knowledge Core RAG system has 4 cost phases:

| Phase | What It Does | Cost Driver |
|---|---|---|
| **1. Document Ingesting** | Parse and embed documents (HR policies, Excel sheets) into a vector DB | Embedding API cost per token |
| **2. Query Processing** | Users ask questions → semantic search retrieves relevant chunks | Embedding cost for query + chunks |
| **3. Agent Respond (LLM)** | Retrieved chunks + system prompt → LLM generates answer | LLM input/output token pricing |
| **4. Storage & Infrastructure** | Vector DB hosting + application server | Per-vector pricing + fixed hosting |

The calculator takes slider/adjustable inputs and outputs:
- **OPEX/month** (USD + VND)
- **Cost per query** and **cost per user**
- **CAPEX** (one-time data ingestion + system development)
- **Phase-by-phase breakdown** with formulas shown
- **Donut chart** of budget allocation
- **3-phase deployment roadmap** for PNJ (UAT → HR Production → Public)

---

## Tech Stack

| Component | Choice |
|---|---|
| **Frontend** | Vanilla HTML/CSS/JS (single file, zero dependencies beyond CDN) |
| **Charts** | [Chart.js 4.4](https://www.chartjs.org/) (CDN — doughnut chart) |
| **Fonts** | IBM Plex Sans + IBM Plex Mono (Google Fonts CDN) |
| **Model Pricing** | Live fetch from `https://openrouter.ai/api/v1/models` |
| **Exchange Rate** | Hardcoded VND/USD rate in `VND_RATE` constant (currently 26,254) |
| **Hosting** | GitHub Pages (static file serving) |

---

## File Structure

```
kb-budget-calculator/
├── index.html    # The entire application (HTML + CSS + JS, ~1213 lines)
├── .gitignore    # Empty
└── README.md     # This file
```

---

## How It Works (for LLMs / developers reading this)

### 1. Model Loading
On page load, the app calls `https://openrouter.ai/api/v1/models`, filters out free/invalid/inappropriate models (image, audio, router models), and populates a searchable `<select>` dropdown. Each model shows its price per 1M input/output tokens.

If the API fails, a hardcoded fallback list of 5 models is used:
- Claude Sonnet 4.6
- Gemini 3.1 Pro
- GPT-5.4 Mini
- DeepSeek V4 Flash
- Qwen3.5 Flash

### 2. Infrastructure Cost (Nội suy tuyến tính)
Uses linear interpolation between two known data points:
- $2,000/month at 100 daily users
- $5,000/month at 5,000 daily users

Formula: `$2000 + (users - 100) / (4900) × $3000`

### 3. Ingest Mode Toggle
Two modes for document ingestion:
- **Pages mode**: User specifies document count, pages per doc, and new docs per month. Assumes 500 tokens per page.
- **Cells mode**: User specifies Excel cell count, max chars per cell (default 32,767), and new cells per month. Tokens estimated as `chars ÷ 2` (Vietnamese text ratio).

Both modes compute `totalChunks = totalTokens ÷ chunkSize`, then `ingestCost = (totalChunks × chunkSize / 1M) × embeddingRate`.

### 4. Query Processing
- `totalQueries = users × requestsPerDay × workingDays`
- `effectiveQueries = totalQueries × (1 − cacheHitRate)`
- Each query retrieves N chunks × chunkSize tokens of context
- Query embedding cost: `effectiveQueries × 50 tokens × embeddingRate`

### 5. LLM Agent Cost
- **Input tokens per query**: `systemPrompt + (chunks × chunkSize) + 50 (query wrapper)`
- **Input cost**: `effectiveQueries × inputTokens × modelInputPrice`
- **Output cost**: `effectiveQueries × outputTokens × modelOutputPrice`

### 6. Storage
- `totalVectors = (docs + newDocs) × chunksPerDoc`
- Cost: `(totalVectors / 1M) × vectorDBRate + serverHosting`

### 7. Preset Scenarios
4 general presets + 3 phase-specific presets bound to `<button>` elements:
- **PNJ tổng (8K)**: 8000 users, 1 req/day, 1455 daily users
- **HRer only**: 200 users, 2 req/day
- **Budget min**: 100 users, 1 req/day, cheapest configs
- **Enterprise**: 5000 users, 500 documents, premium configs
- **GĐ 1 (UAT)**: 2-week test, 100 users, 5 req/day
- **GĐ 2 (HR Production)**: 2 months, 100 HRer, 2 req/day
- **GĐ 3 (Public PNJ)**: ~7 months, 8000 users, 4 req/day

### 8. Phase Timeline Panel
When a GĐ scenario is selected, a dynamic table appears showing all 3 phases side-by-side with editable duration, user count, and requests/day per phase. Changes only affect the timeline calculation (not the main calculator). Total project cost is summed across all phases.

### 9. Dark Mode
Detects `prefers-color-scheme: dark` on load. Toggle button switches between light/dark CSS custom properties.

---

## Key Constants

| Constant | Value | Location |
|---|---|---|
| `VND_RATE` | 26,254 | Line ~590 (hardcoded) |
| `TIER1` | 100 users → $2,000 | Line ~585 |
| `TIER2` | 5,000 users → $5,000 | Line ~586 |
| `TOK_PER_PAGE` | 500 | In `calculate()` function |
| Default model | `anthropic/claude-sonnet-4.6` | Fallback in `fetchModels()` |

---

## How to Update

1. Edit `index.html` locally
2. Commit and push to `master`:
   ```bash
   git add index.html
   git commit -m "Describe your change"
   git push
   ```
3. GitHub Pages auto-deploys within ~60 seconds
4. Verify at `https://drak905.github.io/kb-budget-calculator/`

### Common updates:
- **Exchange rate**: Change `VND_RATE` at line ~590
- **Infrastructure tiers**: Change `TIER1`/`TIER2` at lines ~585–586
- **Embedding model options**: Edit `<select id="sel-embed">` in HTML
- **Vector DB options**: Edit `<select id="sel-vdb">` in HTML
- **Preset scenarios**: Modify the `presets` object in `loadScenario()` function
- **CAPEX development costs**: Edit the hardcoded VND ranges in the CAPEX card HTML

---

## Dependencies (CDN — no install needed)

```html
<!-- Google Fonts -->
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Sans...">

<!-- Chart.js -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js">
```

If these CDNs go down, replace with self-hosted copies or alternative CDN URLs.

---

## Architecture Notes

- **Zero framework**: No React, Vue, or bundler. Pure imperative DOM manipulation via `document.getElementById()`.
- **Reactive pattern**: Sliders sync bidirectionally with number inputs via `syncPair()`. All inputs trigger `calculate()` on change.
- **Chart lifecycle**: A single `Chart.js` doughnut instance is created once in `initChart()` and updated via `donutChart.update()` on each recalculation.
- **State**: All state lives in DOM values. `phaseOverrides` and `phasePresets` are the only JS-side state objects.
- **API call**: Single `fetch()` on page load. No authentication required (OpenRouter public endpoint).
- **CORS**: OpenRouter API allows cross-origin requests from any origin (browser-based fetch works from any hosted domain).
