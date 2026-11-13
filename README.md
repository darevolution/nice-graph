# NICE Appraisals

Knowledge graph and full-text corpus of [NICE Technology Appraisals](https://www.nice.org.uk/guidance/published?ngt=Technology+appraisal+guidance).

826 TAs · 3,307 documents · 555 with structured entity extraction.

**[nice.shoulde.rs](https://nice.shoulde.rs)**

## AI / API access

Point your agent at `https://nice.shoulde.rs/llms.txt` for a machine-readable overview and API guide.

| Endpoint | Description |
|----------|-------------|
| `GET /llms.txt` | Project overview and API guide for AI agents |
| `GET /llms-full.txt` | Full corpus index (826 TAs with document counts) |
| `GET /api/search?q={query}&format=plain` | Full-text search across 3,307 documents |
| `GET /api/corpus/ta{N}/` | Document listing for a TA |
| `GET /api/corpus/ta{N}/{doc}.md` | Raw markdown with `<!-- page N -->` markers |
| `POST /api/chat` | Natural language → SQL → answer (SSE stream) |

## What's in the graph

555 TAs have structured extraction from Final Appraisal Documents:

| Entity | Count |
|--------|-------|
| Drugs | 572 |
| Conditions | 1,298 |
| Methodological decisions | 9,509 (23 categories) |
| Evidence gaps | 3,658 |
| Comparators | 2,769 |
| Clinical trials | 2,206 |
| Cross-references | 952 |
| ICER bands | 926 |
| Economic models | 675 |
| Commercial arrangements | 413 |

Each methodological decision captures the company position, ERG position, and committee preference.

## Pipeline

| Script | Step |
|--------|------|
| `scripts/01_index.py` | Scrape NICE website for all TA listings |
| `scripts/02_download.py` | Download PDFs |
| `scripts/03_extract.py` | PDF → markdown with page markers |
| `scripts/04_chunk.py` | Split into 10-page overlapping windows |
| `scripts/05_tag.py` | AI extraction via Claude Haiku tool calling |
| `scripts/06_resolve.py` | Entity resolution and graph construction |
| `scripts/07_fts_index.py` | Build FTS5 search index |
| `scripts/08_build_downloads.py` | Package corpus archives |

## Run locally

**Pipeline** (rebuilds from scratch):

```bash
uv sync
ANTHROPIC_API_KEY=sk-... uv run python scripts/01_index.py
# ... through 08
```

**Web app** (requires `nice.db` and `data/` from the pipeline):

```bash
cd web
npm install
NUXT_ANTHROPIC_API_KEY=sk-... npx nuxi dev
```

## Ontology

Extraction schema developed over 5 rounds across 50 TAs. Two independent AI agents proposed ontologies from 20 TAs, merged and refined iteratively. 9 entity types, 23 methodological decision categories, 8 ICER bands.

See [`ontology/methods.md`](ontology/methods.md).

## License





Methodology
Structured dataset of NICE Technology Appraisals — the UK's health technology assessments that determine NHS drug funding. 826 appraisals indexed with 3,307 source documents. 555 have structured entity extraction from Final Appraisal Documents.

What's in the graph
Interventions (572 drugs) — generic name, drug class, mechanism, route, duration type
Conditions (1,298) — name, therapeutic area, disease setting, biomarker
Methodological decisions (9,509 across 23 categories) — company position, ERG position, committee preference, ICER impact. Categories include survival extrapolation, utility source, comparator selection, crossover adjustment, model structure, and 18 others.
ICER bands (926) — below £20k, £20–30k, £30–50k, £50–100k, above £100k, dominant, confidential
Clinical trials (2,206) — design, phase, blinding, crossover, generalisability
Comparators (2,769) — type, established practice, committee preferred
Economic models (675) — type, time horizon, health states
Commercial arrangements (413) — PAS, MAA, CAA
Evidence gaps (3,658) — type and description
Cross-references (952) — TA-to-TA citations with relationship type
Pipeline
Scraped NICE website for all TA listings via the Next.js JSON API (894 TAs indexed)
Downloaded 4,522 PDFs (7.3 GB)
Converted to markdown with page-level markers using pymupdf4llm (3,307 files, 380K pages)
Chunked into 10-page windows with 2-page overlap (2,314 FAD chunks)
Extracted entities using Claude Haiku 4.5 with tool calling against a purpose-built ontology. 2,314/2,314 chunks processed, 0 errors.
Normalised enums, deduplicated entities across overlapping chunks, built 14 typed SQLite tables
Built FTS5 full-text search index across all 3,307 documents
Ontology development
The extraction schema was developed iteratively: two independent AI agents each proposed an ontology from 20 diverse TAs, then proposals were merged and refined over 5 rounds on 50 additional TAs. Stabilised at 9 entity types, 23 methodological decision categories, and 8 ICER bands, with a 3.5% “other” rate for decision categories. See ontology/methods.md.

Coverage and limitations
826 TAs have source documents; 555 have structured extraction
Extraction targets Final Appraisal Documents only — not ERG reports, scope comments, or committee papers
Numerical results (ICERs, QALYs, costs, hazard ratios) are deliberately not extracted — stored as categorical bands instead
Older TAs (pre-2006) have lower extraction quality due to different document formats
Extraction uses AI and is not perfect — verify against source documents for critical decisions
API
Endpoint	Description
/llms.txt	Machine-readable project overview and API guide
/llms-full.txt	Full corpus index (all TAs with document counts)
/api/search?q=...&format=plain	Full-text search
/api/corpus/ta{N}/	Document listing for a TA
/api/corpus/ta{N}/{doc}.md	Raw document markdown
POST /api/chat	Natural language → SQL → answer (SSE stream)
Source
GitHub · Data from NICE · Built by Abundance Decision Systems

MIT
