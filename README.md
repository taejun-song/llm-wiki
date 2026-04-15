# llm-wiki — starter chassis for an LLM-maintained knowledge base

A minimal, domain-agnostic template for running an **LLM-wiki** — a persistent, compounding markdown knowledge base that an LLM agent reads, updates, and cross-links on your behalf as you work through a long-running research or learning project.

Fork this repo, point your LLM agent at it, and start ingesting sources.

---

## What an LLM-wiki is

A design pattern for personal knowledge bases in which an LLM **incrementally builds and maintains a persistent, interlinked wiki** that sits between you and a growing collection of raw sources — as a deliberate alternative to stateless retrieval (RAG-style) on every query.

Three layers:

| Layer | Path | Who owns it |
|---|---|---|
| **Raw sources** | `raw/` (with `raw/articles/`, `raw/papers/`, `raw/assets/`) | You drop sources here. The agent treats this layer as read-only. |
| **Wiki pages** | `concepts/`, `entities/`, `summaries/`, `comparisons/`, `syntheses/` | Agent-owned. Each markdown page has YAML frontmatter, a one-sentence summary as its first paragraph, and `[[wiki-link]]` cross-references to other pages. |
| **Special files** | `index.md`, `log.md`, `CLAUDE.md` | Agent-maintained. `CLAUDE.md` is the operating contract (see below); `index.md` is regenerated after every ingest; `log.md` is an append-only activity log. |

The key move that distinguishes this from a plain Obsidian vault: **the agent is bound by a schema document (`CLAUDE.md`) that specifies exactly how to read sources, when to create new pages, what frontmatter to include, how to cross-link, and how to lint the result.** Without the schema it's just notes; with the schema it's a compounding artifact.

## Why not just RAG?

RAG re-derives answers from raw chunks on every query. An LLM-wiki compiles knowledge once into reusable pages and then reads the pages. The tradeoff:

| | RAG | LLM-wiki |
|---|---|---|
| Upfront cost | None | Every source needs a planned ingest |
| Per-query cost | Retrieval + read + generate | Cheap read of 1–5 pages |
| Graph structure | Flat chunks, no cross-refs | Typed pages with bidirectional links |
| Best for | One-shot Q&A over a fixed corpus | Long-running research with dozens–hundreds of sources that share entities |

Use RAG when your corpus is flat and queries are one-shot. Use an LLM-wiki when you're going to come back to the same material over months and want the graph structure and compounding effect.

## Repository layout

```text
.
├── README.md          — this file
├── CLAUDE.md          — the operating contract the agent follows (read this second)
├── index.md           — regenerated index of every page, regenerated after each ingest
├── log.md             — append-only activity log
├── concepts/          — reusable ideas (e.g. "chain-of-thought", "long-context inference")
├── entities/          — named things (people, orgs, models, papers, datasets, places)
├── summaries/         — one page per raw source, with distilled takeaways
├── comparisons/       — side-by-sides of two or more concepts or entities
├── syntheses/         — cross-source views that don't fit into any single concept page
└── raw/
    ├── articles/      — markdown dumps of web articles
    ├── papers/        — PDFs
    └── assets/        — images, CSVs, JSONs, audio transcripts
```

All five typed subdirectories start empty with a `.gitkeep` placeholder — you populate them as you ingest sources.

## How to fork this for a new domain

1. **Clone the template.**
   ```bash
   git clone git@github.com:taejun-song/llm-wiki.git my-new-domain-wiki
   cd my-new-domain-wiki
   rm -rf .git && git init && git add . && git commit -m "bootstrap wiki from starter"
   ```
2. **Rename it in the two places where it matters**: the `# Title` in `index.md` (if it exists) and the H1 of `README.md`. Nothing else has a hard-coded name.
3. **Read `CLAUDE.md`.** It's the operating contract the agent follows. Skim it once; you don't need to memorize it.
4. **Drop your first source** into `raw/articles/` or `raw/papers/`.
5. **Ask your agent to ingest it.** The agent will propose a plan (3–7 key takeaways, pages to create, pages to update, cross-links to add), wait for your approval, and then apply the writes in a strict order with an automatic index regeneration and log entry at the end.
6. **Repeat.** After ~10 ingests the wiki's shape will stabilise; after ~30 it becomes genuinely useful for answering cross-source questions.

## The ingest-discuss-apply workflow

Every ingest follows this loop:

```text
┌────────┐      ┌────────┐     ┌──────────┐     ┌───────┐     ┌────────┐
│  READ  │ ───► │  PLAN  │───► │ DISCUSS  │───► │ APPLY │───► │ REPORT │
│ source │      │ in chat│     │ w/ user  │     │ writes│     │ result │
└────────┘      └────────┘     └──────────┘     └───────┘     └────────┘
```

1. **READ** — the agent reads the raw source (PDF, article, image, transcript).
2. **PLAN** — proposes 3–7 key takeaways, new pages to create, existing pages to update, and cross-links to add.
3. **DISCUSS** — you approve, redirect, or skip. No writes happen yet.
4. **APPLY** — on your go-ahead, the agent writes new pages → updates existing pages → creates the source-summary → regenerates `index.md` → appends to `log.md`, in that exact order.
5. **REPORT** — you get a summary of what was created, what was updated, how many links were added, and any contradictions recorded.

See `CLAUDE.md` §§ 6.1–6.5 for the full workflows (INGEST, QUERY, LINT, INDEX REGENERATION, LOG ENTRIES).

## Design references

The LLM-wiki pattern was articulated publicly by Andrej Karpathy in the gist at <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>. A sibling pattern — applying the same "compounding artifact + agent-schema document" meta-idea to training scripts instead of markdown — is Karpathy's `autoresearch` repo at <https://github.com/karpathy/autoresearch>. Both are worth reading before you adapt this template for a demanding domain.

## Related templates in this workspace

- `vlm-wiki/` — a concrete instance of this chassis specialised for vision-language-model research (concepts, entities, summaries, syntheses about OpenEQA, DynaMem, MemoryVLA, EchoVLA, LLM-wiki meta, etc.).
- `bio-wiki/`, `math-wiki/`, `eco-standard-wiki/` — other domain instances.
- `llm-wiki/` — **this repo**: the empty chassis that all of the above were forked from.

## License

MIT
