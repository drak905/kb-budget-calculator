# Knowledge Core Budget Calculator

Single-file web calculator for estimating AI system CAPEX and OPEX for PNJ deployment planning.

The app now has three top-level calculator categories:

1. **Knowledge Core** - existing RAG CAPEX + OPEX calculator.
2. **Contract Review** - new OPEX-only calculator for AI-powered contract review.
3. **Audio Analysis** - placeholder for a future OPEX calculator.

- **Live site**: `https://drak905.github.io/kb-budget-calculator/`
- **Hosting**: GitHub Pages
- **Frontend**: standalone `index.html`
- **Backend**: none

---

## What The App Estimates

### Knowledge Core

The Knowledge Core calculator estimates the cost of running a RAG system for organizational knowledge access.

It covers:

| Phase | What It Does | Cost Driver |
|---|---|---|
| Document Ingesting | Parse and embed documents into a vector database | Embedding API cost per token |
| Query Processing | Embed user questions and retrieve relevant chunks | Query embedding and retrieval context |
| Agent Respond | Send prompt + retrieved context to an LLM | LLM input/output token pricing |
| Storage & Infrastructure | Host vector DB and application server | Vector DB pricing + hosting |

Outputs include:

- Monthly OPEX in USD and VND.
- Cost per query.
- Cost per user.
- One-time CAPEX for initial ingest.
- Phase-by-phase cost table.
- Doughnut chart budget breakdown.
- PNJ deployment timeline presets.

### Contract Review

The Contract Review calculator estimates monthly OPEX for an AI system that reviews contracts against template contracts and review rules.

User stories covered:

- Users upload template contracts.
- The AI compares a real contract against the template contract.
- The AI identifies contract section status such as missing, excess, needs change, unsuitable, or appropriate.
- The AI uses a contract review system prompt to suggest adjustments that maximize company benefit.

Cost components:

| Component | What It Represents | Formula Basis |
|---|---|---|
| Infrastructure | VPS cost for daily users | Capped doubling tier model |
| LLM Input | Template + real contract + review rules | Input tokens per review |
| LLM Output | Review result and suggestions | Output tokens per review |

Contract Review does not include embedding or vector DB cost in this version.

### Audio Analysis

Audio Analysis is currently a non-functional placeholder. It is reserved for a future OPEX calculator.

---

## Contract Review Formula

### AI Service

The calculator assumes the template contract is included in every review request.

```text
template_tokens = template_pages * 500
contract_tokens = contract_pages * 500
input_tokens_per_review = template_tokens + contract_tokens + system_prompt_tokens

llm_input_cost =
  contracts_reviewed_per_month
  * input_tokens_per_review
  / 1,000,000
  * selected_model_input_price

llm_output_cost =
  contracts_reviewed_per_month
  * output_tokens_per_review
  / 1,000,000
  * selected_model_output_price
```

### Infrastructure

Contract Review VPS cost doubles every 20 daily active users and is capped at `$3,200/month`.

```text
raw_tier = max(0, ceil(daily_users / 20) - 1)
tier = min(raw_tier, 4)
vps_cost = 200 * 2^tier
```

Tier examples:

| Daily Users | VPS Cost |
|---:|---:|
| 1-20 | $200/month |
| 21-40 | $400/month |
| 41-60 | $800/month |
| 61-80 | $1,600/month |
| 81+ | $3,200/month |

---

## Tech Stack

| Component | Choice |
|---|---|
| Frontend | Vanilla HTML, CSS, and JavaScript |
| Charts | Chart.js 4.4 from CDN |
| Fonts | IBM Plex Sans and IBM Plex Mono from Google Fonts |
| LLM pricing | Live fetch from `https://openrouter.ai/api/v1/models` |
| Embedding models | Static 26-model catalog |
| Exchange rate | Live fetch from `https://open.er-api.com/v6/latest/USD` |
| Hosting | GitHub Pages |

No package install or build step is required.

---

## File Structure

```text
kb-budget-calculator/
├── index.html          # Entire app: HTML, CSS, and JavaScript
├── README.md           # Project documentation
└── .gitignore
```

Additional local planning and changelog files may exist during development, but the app itself runs from `index.html`.

---

## How It Works

### Model Loading

On page load, the app fetches live model pricing from OpenRouter:

```text
https://openrouter.ai/api/v1/models
```

The app filters out invalid, free, image, audio, router, and unsuitable model entries, then populates the shared `LLM Model` selector.

If OpenRouter fails, the app falls back to a small hardcoded model list.

### Shared LLM Selector

The LLM selector is shared across:

- Knowledge Core.
- Contract Review.

Changing the selected LLM recalculates the currently active tab.

### Embedding Models

Embedding models are loaded from a static catalog because the OpenRouter embeddings endpoint does not currently support browser CORS.

The embedding selector is only used by Knowledge Core.

### Tab System

The UI uses `data-section` attributes to show and hide calculator sections:

- `data-section="shared"` is always visible in the sidebar.
- `data-section="knowledge-core"` is visible on the Knowledge Core tab.
- `data-section="contract-review"` is visible on the Contract Review tab.
- `data-section="audio-analysis"` is visible on the Audio Analysis tab.

`switchTab(tab)` controls visibility and triggers the correct recalculation function.

### Chart Lifecycle

The app uses two Chart.js doughnut chart instances:

- `donutChart` for Knowledge Core.
- `donutChartCR` for Contract Review.

Both charts update when costs are recalculated.

---

## Key Constants

| Constant / Concept | Value |
|---|---|
| `VND_RATE` fallback | 26,254 VND/USD |
| Knowledge Core infra tier 1 | 100 users -> $2,000/month |
| Knowledge Core infra tier 2 | 5,000 users -> $5,000/month |
| Tokens per page | 500 |
| Contract Review base VPS | $200/month |
| Contract Review VPS cap | $3,200/month |
| Contract Review default template pages | 10 |
| Contract Review default contracts/month | 20 |
| Contract Review default real contract pages | 15 |
| Contract Review default system prompt | 800 tokens |
| Contract Review default output | 800 tokens/review |

---

## How To Update

1. Edit `index.html`.
2. Test locally by opening `index.html` in a browser.
3. Commit and push to `master`:

```bash
git add index.html README.md
git commit -m "Update calculator"
git push origin master
```

4. GitHub Pages should auto-deploy shortly after the push.
5. Verify the live site:

```text
https://drak905.github.io/kb-budget-calculator/
```

---

## Common Maintenance Tasks

- Update OpenRouter fallback models in `fetchModels()`.
- Update embedding catalog in `fetchEmbedModels()`.
- Update Knowledge Core infrastructure tiers in `TIER1` and `TIER2`.
- Update Contract Review VPS logic in `calcInfraContractReview()`.
- Update Contract Review inputs and formulas in `calculateContractReview()`.
- Update exchange-rate fallback in `VND_RATE`.
- Update PNJ scenario presets in `loadScenario()`.

---

## Architecture Notes

- The app intentionally remains a single static HTML file.
- There is no framework, bundler, backend, or database.
- All state is stored in DOM values plus small JavaScript state variables.
- Slider and number input pairs are synchronized in JavaScript.
- Knowledge Core uses `calculate()`.
- Contract Review uses `calculateContractReview()`.
- Shared LLM changes go through `onModelChange()`.
- Dark mode updates both chart instances.
