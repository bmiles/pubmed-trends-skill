---
name: pubmed-trends
description: >
  Query PubMed publication trends via x402 micropayments to identify emerging research fields,
  compare therapeutic areas, and detect publication spikes. Use when analyzing biotech/pharma
  landscapes, writing grant applications, evaluating drug targets, or researching scientific trends.
  Requires the x402 skill from coinbase/agentic-wallet-skills for payment commands.
---

# PubMed Trends API

Pay-per-query research intelligence from 37M+ PubMed articles. No API keys, no subscriptions — just USDC micropayments on Base via x402.

**Base URL:** `https://pubmed.sekgen.xyz`

**Companion skill:** Install `x402` from `coinbase/agentic-wallet-skills` for payment commands.

## Endpoints

### GET /api/v1/snapshot — $0.01

Single-year publication count. Cheapest option for quick lookups.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `query` | Yes | — | PubMed search query |
| `year` | No | current year | Year to query |

```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/snapshot?query=CRISPR&year=2025'
```

Returns `count`, `total_pubmed`, `proportion` (share of all PubMed articles).

### GET /api/v1/trends — $0.01

Year-by-year publication counts with growth analytics over a range.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `query` | Yes | — | PubMed search query |
| `start_year` | No | current - 10 | First year |
| `end_year` | No | current year | Last year |

```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/trends?query=GLP-1+agonist&start_year=2015'
```

Returns yearly `count`, `yoy_growth`, plus analytics: `cagr`, `trend_direction` (growing/declining/stable/emerging), `peak_year`, `acceleration`.

### GET /api/v1/compare — $0.02

Compare publication trends across 2–5 search terms. Identifies which field is growing fastest and detects crossover points.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `terms` | Yes | — | Comma-separated search terms (2–5) |
| `start_year` | No | current - 10 | First year |
| `end_year` | No | current year | Last year |

```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/compare?terms=CAR-T,bispecific+antibodies,antibody+drug+conjugate'
```

Returns per-term series with `cagr` and `trend_direction`, plus `ranking_by_growth` and `crossover_years` (when one term overtook another).

### GET /api/v1/pulse — $0.02

Real-time publication pulse with 7d/30d/90d windows and year-over-year change.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `query` | Yes | — | PubMed search query |

```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/pulse?query=prime+editing'
```

Returns `current` and `year_ago` counts for 7d/30d/90d windows, plus `change` (fractional, e.g. 0.35 = 35% increase YoY).

### GET /api/v1/hottest — $0.05

Hottest topics in a category ranked by weekly spike ratio (last 7d vs 30d average).

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `category` | Yes | — | See categories below |

```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/hottest?category=oncology'
```

Returns topics sorted by `spike_ratio`. Values >1.0 mean more activity than usual this week.

### GET /api/v1/emerging — $0.05

Scan a category for topics with accelerating publication rates.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `category` | Yes | — | See categories below |
| `window_years` | No | 3 | Years to evaluate growth |
| `min_growth_rate` | No | 1.0 | Minimum CAGR threshold (1.0 = 100%) |
| `limit` | No | 10 | Max results |

```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/emerging?category=gene+therapy&min_growth_rate=0.5'
```

Returns topics meeting the growth threshold, sorted by `recent_cagr`.

## Categories

Used by `/hottest` and `/emerging`:

- **oncology** — CAR-T, bispecific antibodies, ADCs, checkpoint inhibitors, liquid biopsy, KRAS inhibitors, PARP inhibitors, radiopharmaceuticals, cancer vaccines, spatial transcriptomics
- **neuroscience** — anti-amyloid antibodies, brain-computer interfaces, psychedelic therapy, GLP-1 neurodegeneration, antisense oligonucleotides, deep brain stimulation
- **immunology** — mRNA vaccines, CAR-T autoimmune, JAK inhibitors, regulatory T cells, microbiome immunotherapy, complement inhibitors
- **gene therapy** — CRISPR, base editing, prime editing, AAV gene therapy, lipid nanoparticle mRNA, epigenome editing, RNA interference
- **cardiology** — PCSK9 inhibitors, SGLT2 inhibitors, cardiac gene therapy, lipoprotein(a) inhibitors, AI cardiac imaging, wearable ECG

## Free Endpoints

These do not require payment:

- `GET /` — Service info and pricing
- `GET /health` — Health check
- `GET /docs` — Interactive documentation (HTML, or Markdown with `Accept: text/markdown`)
- `GET /llms.txt` — Machine-readable documentation

## Example Workflows

**Quick fact check ($0.01):**
> "How much CRISPR research was published in 2025?"
```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/snapshot?query=CRISPR&year=2025'
```

**Technology race ($0.02):**
> "Is base editing overtaking traditional CRISPR?"
```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/compare?terms=CRISPR+gene+editing,base+editing,prime+editing&start_year=2018'
```

**Weekly research briefing ($0.10):**
> "What's spiking in oncology and neuroscience right now?"
```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/hottest?category=oncology'
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/hottest?category=neuroscience'
```

**Drug target validation ($0.03):**
> "Is GLP-1 for neurodegeneration actually gaining traction?"
```bash
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/trends?query=GLP-1+agonist+neurodegeneration&start_year=2018'
awal x402 pay 'https://pubmed.sekgen.xyz/api/v1/pulse?query=GLP-1+agonist+neurodegeneration'
```

## Error Handling

- **402** — Payment required. Ensure you're authenticated (`awal auth login`) with USDC balance.
- **400** — Invalid parameters. Check required fields and constraints (max 50-year range, 2–5 terms for compare).
- **502** — Upstream NCBI error. Retry after a moment.
