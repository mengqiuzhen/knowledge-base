# LLM Wiki Methodology

> Sources: Andrej Karpathy, 2026-04-02
> Raw: [Karpathy LLM Wiki Gist](../../raw/llm-wiki/2026-04-02-karpathy-llm-wiki.md)

## Overview

LLM Wiki is a knowledge management pattern proposed by Andrej Karpathy in April 2026. Instead of traditional RAG (retrieve at query time), an LLM acts as a **compiler** that incrementally builds and maintains a persistent, structured wiki of interlinked markdown files. The human curates sources and asks questions; the LLM does all the summarizing, cross-referencing, filing, and bookkeeping. The result is a knowledge base that compounds over time — every new source makes the wiki richer, and cross-references are pre-built rather than recomputed on every query.

## Core Insight: Compiler vs Interpreter

The key metaphor: traditional RAG is an **interpreter** — it re-evaluates raw documents from scratch on every query. LLM Wiki is a **compiler** — knowledge is extracted, structured, and cross-linked once at ingest time, then kept current.

| Dimension | Traditional RAG | LLM Wiki |
|-----------|----------------|----------|
| Knowledge processing | Query time (runtime) | Ingest time (compile time) |
| Accumulation | None — starts fresh each query | Compounds — wiki grows richer |
| Cross-references | Computed ad-hoc per query | Pre-built and maintained |
| Output | Ephemeral chat answer | Persistent markdown files |
| Infrastructure | Vector DB + embedding model | Filesystem + text editor |

## Architecture: Three Layers

### 1. Raw Sources (`raw/`)
Immutable source documents — articles, papers, images, data files. The LLM reads from them but never modifies them. This is the source of truth.

### 2. Wiki (`wiki/`)
LLM-generated and LLM-maintained markdown files: summaries, entity pages, concept pages, comparisons, synthesis. The LLM owns this layer entirely — creating pages, updating them with new sources, maintaining cross-references. The human reads it; the LLM writes it.

### 3. Schema (`SKILL.md` / `CLAUDE.md`)
The configuration document that tells the LLM how the wiki is structured, what conventions to follow, and what workflows to use. This is what transforms the LLM from a generic chatbot into a disciplined wiki maintainer. It co-evolves between human and LLM over time.

## Operations

### Ingest
Adding a new source triggers a multi-step cascade:
1. LLM reads the source and saves it to `raw/`
2. Discusses key takeaways with the human
3. Writes/updates summary pages in the wiki
4. Updates the index
5. Updates relevant entity and concept pages across the wiki
6. Appends an entry to the log

A single source may touch **10-15 wiki pages**. Ingest can be done one-at-a-time with human guidance, or batch-processed with less supervision.

### Query
The LLM searches the wiki (starting from `index.md`), reads relevant pages, and synthesizes an answer with citations. Answers can take multiple forms: markdown pages, comparison tables, slide decks (Marp), charts, canvases. High-quality answers should be **archived back into the wiki** — they represent valuable insights that shouldn't disappear into chat history.

### Lint
Periodic health checks of the wiki. The LLM looks for:
- Contradictions between pages
- Stale claims superseded by newer sources
- Orphan pages with no inbound links
- Important concepts mentioned but lacking dedicated pages
- Missing cross-references
- Knowledge gaps that could be filled by web search

## Navigation: index.md and log.md

- **`wiki/index.md`**: A content-oriented catalog — every page listed with link, one-line summary, and optional metadata. Organized by category. The LLM reads this first to locate relevant pages for queries. Works at moderate scale (~100 sources, hundreds of pages) without vector databases.
- **`wiki/log.md`**: A chronological, append-only record of all operations. Consistent entry prefixes (e.g., `## [2026-04-02] ingest | Title`) make it parseable with simple tools like `grep`.

## Use Cases

- **Personal knowledge management**: goals, health, psychology, journaling
- **Research**: deep-diving a topic over weeks/months with evolving thesis
- **Reading books**: chapter-by-chapter companion wiki (characters, themes, plot)
- **Business/team**: internal wiki fed by meetings, documents, calls
- **Other**: competitive analysis, due diligence, trip planning, course notes, hobby deep-dives

## Tools Mentioned

- **Obsidian**: The "IDE" for browsing the wiki — graph view, backlinks, plugins
- **Obsidian Web Clipper**: Browser extension to convert web articles to markdown
- **Marp**: Markdown-based slide deck format (Obsidian plugin available)
- **Dataview**: Obsidian plugin for querying YAML frontmatter across pages
- **qmd**: Local hybrid search engine (BM25 + vector) for markdown files, with CLI and MCP server

## Why It Works

Maintaining a knowledge base manually fails because the **bookkeeping burden grows faster than the value** — updating cross-references, keeping summaries current, noting contradictions. LLMs don't get bored, don't forget, and can update 15 files in one pass. The maintenance cost drops to near zero, so the wiki actually stays maintained.

The idea traces back to Vannevar Bush's **Memex** (1945) — a personal, curated knowledge store with associative trails. Bush envisioned private, actively curated knowledge where connections between documents are as valuable as the documents themselves. The missing piece was maintenance labor — which LLMs now supply.

## See Also

- [SKILL.md Schema](../../SKILL.md) — this vault's operational schema
