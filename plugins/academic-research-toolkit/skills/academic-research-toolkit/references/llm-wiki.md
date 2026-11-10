# LLM Wiki — Karpathy Pattern Knowledge Base

Build and maintain a persistent, compounding knowledge base as interlinked markdown files.

## Structure

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
├── concepts/          # Concept/topic pages
├── comparisons/       # Side-by-side analyses
└── queries/           # Filed query results
```

## Core Operations

### Ingest
1. Capture raw source (URL → web_extract, save to raw/)
2. Discuss takeaways with user
3. Check existing pages (index.md + search)
4. Write/update wiki pages with [[wikilinks]]
5. Update index.md and log.md

### Query
1. Read index.md for relevant pages
2. search_files for key terms
3. Synthesize answer citing wiki pages
4. File valuable answers as queries/ or comparisons/

### Lint
- Orphan pages (no inbound links)
- Broken wikilinks
- Frontmatter validation
- Source drift detection

## Obsidian Integration

The wiki directory works as an Obsidian vault:
- `[[wikilinks]]` render as clickable links
- YAML frontmatter powers Dataview queries
- Set attachment folder to `raw/assets/`