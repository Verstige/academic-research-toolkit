# Academic Research Toolkit: Papers, Citations, Feeds

> Search arXiv and Semantic Scholar, manage a wiki knowledge base, write papers with a full BibTeX pipeline, and monitor the feeds that matter for your field.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg) ![Claude Code](https://img.shields.io/badge/Claude_Code-skill-blueviolet.svg) [![Available on Agensi](https://img.shields.io/badge/Agensi-%2411.99-ff5a5f.svg)](https://agensi.io/skills/academic-research-toolkit-papers-citations-feeds)

## What it does

Research papers, citations, and feeds from one workflow: arXiv search, an LLM Wiki knowledge base, a research-paper tool, and grounded citations with a source for every claim.

Complete research workflow for academic papers, knowledge bases, and blog monitoring. Covers arXiv search, LLM Wiki management, paper writing, and RSS feed tracking.

## The failure it prevents

Each reference file is one specific failure: the symptom, the actual cause, and the fix. The ones that come up most often:

- arXiv — Academic Paper Search
- Blogwatcher — RSS/Atom Feed Monitoring
- LLM Wiki — Karpathy Pattern Knowledge Base
- SEC EDGAR Research — Public Company Financials
- Web Infrastructure Audit — Methodology

## When to use it

- search arXiv and the literature for prior work
- write a paper with a working citation pipeline
- keep a research wiki and monitor the feeds that matter

## What's inside

```
plugins/
└── academic-research-toolkit/
    ├── .claude-plugin/plugin.json
    └── skills/academic-research-toolkit/
        ├── SKILL.md
        ├── references/            (5 files)
        │   └── arxiv.md
        │   └── blogwatcher.md
        │   └── llm-wiki.md
        │   └── sec-edgar.md
        │   └── web-infra-audit.md
```

## Install

Add this repo as a Claude Code marketplace, then install the skill:

```bash
claude plugin marketplace add Verstige/academic-research-toolkit
claude plugin install academic-research-toolkit@academic-research-toolkit
```

Or copy the skill straight into your skills directory:

```bash
git clone https://github.com/Verstige/academic-research-toolkit.git
cp -R academic-research-toolkit/plugins/academic-research-toolkit/skills/academic-research-toolkit ~/.claude/skills/
```

## Usage

The skill activates on the work it describes. Ask for the outcome, and it
supplies the method:

```
> "search arXiv and the literature for prior work"
```

## Reference material

Loaded on demand — the skill only reads what the task needs.

| File | Covers |
| --- | --- |
| `arxiv.md` | arXiv — Academic Paper Search |
| `blogwatcher.md` | Blogwatcher — RSS/Atom Feed Monitoring |
| `llm-wiki.md` | LLM Wiki — Karpathy Pattern Knowledge Base |
| `sec-edgar.md` | SEC EDGAR Research — Public Company Financials |
| `web-infra-audit.md` | Web Infrastructure Audit — Methodology |

## License

MIT. See [LICENSE](LICENSE).

---

**Also available as a paid skill on Agensi** — $11.99. Install it with one command, no clone required:

[agensi.io/skills/academic-research-toolkit-papers-citations-feeds](https://agensi.io/skills/academic-research-toolkit-papers-citations-feeds)

Part of the [Verstige](https://github.com/Verstige) skills portfolio. More at [github.com/Verstige](https://github.com/Verstige).
