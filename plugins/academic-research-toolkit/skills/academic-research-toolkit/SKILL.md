---
name: academic-research-toolkit
description: "Research papers, citations, and feeds from one workflow: arXiv search, an LLM Wiki knowledge base, a research-paper tool, and grounded citations with a source for every claim."
version: 1.0.0
license: MIT
author: "the maintainers"
---
# Research Tools — Academic + Technical Research

Complete research workflow for academic papers, knowledge bases, and blog monitoring. Covers arXiv search, LLM Wiki management, paper writing, and RSS feed tracking.

---

## arXiv Research

### Search Papers

```bash
# Basic search
curl -s "https://export.arxiv.org/api/query?search_query=all:GRPO+reinforcement+learning&max_results=5"

# With date sort
curl -s "https://export.arxiv.org/api/query?search_query=all:transformer&sortBy=submittedDate&sortOrder=descending&max_results=10"
```

### Search Query Syntax

| Prefix | Searches | Example |
|--------|----------|---------|
| `all:` | All fields | `all:transformer+attention` |
| `ti:` | Title | `ti:large+language+models` |
| `au:` | Author | `au:vaswani` |
| `abs:` | Abstract | `abs:reinforcement+learning` |
| `cat:` | Category | `cat:cs.AI` |

### Fetch Paper

```bash
# By ID
curl -s "https://export.arxiv.org/api/query?id_list=2402.03300"

# Multiple papers
curl -s "https://export.arxiv.org/api/query?id_list=2402.03300,2401.12345"
```

### Read Paper Content

```bash
# Abstract (fast)
web_extract(urls=["https://arxiv.org/abs/2402.03300"])

# Full paper PDF
web_extract(urls=["https://arxiv.org/pdf/2402.03300"])
```

### Semantic Scholar (Citations)

```bash
# Paper details
curl -s "https://api.semanticscholar.org/graph/v1/paper/arXiv:2402.03300?fields=title,authors,citationCount,year"

# Citations of a paper
curl -s "https://api.semanticscholar.org/graph/v1/paper/arXiv:2402.03300/citations?fields=title,authors,year&limit=10"

# References from a paper
curl -s "https://api.semanticscholar.org/graph/v1/paper/arXiv:2402.03300/references?fields=title,citationCount&limit=10"
```

---

## LLM Wiki (Karpathy Pattern)

### Wiki Structure

```
wiki/
├── SCHEMA.md           # Conventions, structure, tag taxonomy
├── index.md            # Content catalog with one-line summaries
├── log.md              # Chronological action log
├── raw/                # Immutable source material
│   ├── articles/       # Web articles, clippings
│   ├── papers/         # PDFs, arxiv papers
│   └── transcripts/    # Meeting notes, interviews
├── entities/           # Entity pages (people, orgs, products)
├── concepts/           # Concept/topic pages
├── comparisons/        # Side-by-side analyses
└── queries/            # Filed query results
```

### Core Operations

**Ingest:**
1. Capture raw source (URL → web_extract, save to raw/)
2. Discuss takeaways with user
3. Check existing pages (index.md + search)
4. Write/update wiki pages with [[wikilinks]]
5. Update index.md and log.md

**Query:**
1. Read index.md for relevant pages
2. search_files for key terms
3. Synthesize answer citing wiki pages
4. File valuable answers as queries/ or comparisons/

**Lint:**
- Orphan pages (no inbound links)
- Broken wikilinks
- Frontmatter validation
- Source drift detection

### Obsidian Integration

The wiki directory works as an Obsidian vault:
- `[[wikilinks]]` render as clickable links
- YAML frontmatter powers Dataview queries
- Set attachment folder to `raw/assets/`

---

## Research Paper Writing

### Pipeline (NeurIPS/ICML/ICLR)

1. **Design** — Research question, hypothesis, methodology
2. **Experiments** — Run experiments, collect metrics
3. **Analysis** — Statistical analysis, significance testing
4. **Writing** — LaTeX paper with figures, tables
5. **Review** — Self-review, external review
6. **Submit** — Camera-ready, format checking

### Citation Management

```bash
# Generate BibTeX
curl -s "https://export.arxiv.org/api/query?id_list=1706.03762" | python3 -c "
import sys, xml.etree.ElementTree as ET
ns = {'a': 'http://www.w3.org/2005/Atom'}
root = ET.parse(sys.stdin).getroot()
entry = root.find('a:entry', ns)
title = entry.find('a:title', ns).text.strip().replace('\n', ' ')
authors = ' and '.join(a.find('a:name', ns).text for a in entry.findall('a:author', ns))
year = entry.find('a:published', ns).text[:4]
raw_id = entry.find('a:id', ns).text.strip().split('/abs/')[-1]
print(f'@article{{{entry.find(\"a:author\", ns).find(\"a:name\", ns).text.split()[-1]}{year}_{raw_id.replace(\".\",\"\")},')
print(f'  title = {{{title}}},')
print(f'  author = {{{authors}}},')
print(f'  year = {{{year}}},')
print(f'  eprint = {{{raw_id}}},')
print(f'  archivePrefix = {{arXiv}}')
print('}')
"
```

---

## Blogwatcher (RSS/Atom Monitoring)

### Setup

```bash
# Install
go install github.com/JulienTant/blogwatcher-cli/cmd/blogwatcher-cli@latest

# Docker
docker run --rm -v blogwatcher-cli:/data ghcr.io/julientant/blogwatcher-cli
```

### Commands

```bash
# Add blog
blogwatcher-cli add "My Blog" https://example.com

# Scan
blogwatcher-cli scan

# List articles
blogwatcher-cli articles

# Mark read
blogwatcher-cli read 1

# Import OPML
blogwatcher-cli import subscriptions.opml
```

---

## Related Skills

- **`arxiv`** — Detailed arXiv API usage, Semantic Scholar integration, BibTeX generation
- **`llm-wiki`** — Full Karpathy LLM Wiki implementation with schema, operations, Obsidian sync
- **`research-paper-writing`** — ML paper pipeline (design → experiments → submit)
- **`blogwatcher`** — RSS/Atom feed monitoring with blogwatcher-cli

---

## Complete Research Workflow

```
1. Discover    → python the arXiv API "topic" --sort date --max 10
2. Assess      → curl semantic scholar for citation counts
3. Read        → web_extract abstract or full paper
4. Ingest      → save to wiki/raw/, write wiki pages
5. Analyze     → find related work, get recommendations
6. Track       → blogwatcher-cli for ongoing monitoring
```

## SEC EDGAR Research

For public company financial research (ticker, CIK, or company name → revenue, net income, assets, filings):

→ See `references/sec-edgar.md`

---

## Web Infrastructure Audit

For auditing a web platform's full technology stack, API surface, and business systems — brokers, SaaS platforms, trading systems, HYIP platforms.

→ See `references/web-infra-audit.md`

Covers: DNS enumeration, SSL cert analysis, JS bundle extraction, SPA route discovery, Supabase backend probing, edge function fingerprinting, SSE detection, business system mapping, white-label pattern detection, and competitive analysis.

---

## Rate Limits

| API | Rate | Auth |
|-----|------|------|
| arXiv | ~1 req / 3 seconds | None |
| Semantic Scholar | 1 req / second | None (100/sec with key) |