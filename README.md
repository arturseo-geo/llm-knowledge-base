# LLM Knowledge Base

> Built by **[Artur Ferreira](https://github.com/arturseo-geo)** @ **The GEO Lab** · [𝕏 @TheGEO_Lab](https://x.com/TheGEO_Lab) · [LinkedIn](https://linkedin.com/in/arturgeo) · [Reddit](https://www.reddit.com/user/Alternative_Teach_74/)

![Version](https://img.shields.io/badge/version-1.1.0-blue)
![Licence](https://img.shields.io/badge/licence-MIT-green)
![Schema](https://img.shields.io/badge/schema-AGENTS.md-blueviolet)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen)](https://github.com/arturseo-geo/llm-knowledge-base/blob/main/CONTRIBUTING.md)

**A schema and workflow for LLM-compiled personal knowledge bases.**

Inspired by Andrej Karpathy's April 2026 post on using LLMs to build and maintain markdown wikis, this repo formalises the pattern into a reusable, versioned schema — including a structured learning layer that the original workflow leaves out.

The core idea: you collect raw source material. The LLM compiles it into a structured wiki. You never edit the wiki manually. You just learn from it.

---

## What this repo contains

| File | Purpose |
|---|---|
| [`AGENTS.md`](./AGENTS.md) | The schema spec — every LLM agent operating on your wiki reads this first |
| [`templates/`](./templates/) | Starter templates for each file type in the schema |
| [`examples/`](./examples/) | A worked example wiki (AI alignment topic, fully populated) |
| [`docs/why-not-rag.md`](./docs/why-not-rag.md) | When to use a compiled wiki vs a vector RAG stack |
| [`docs/learning-layer.md`](./docs/learning-layer.md) | How the spaced repetition and gap tracking system works |
| [`docs/contamination-mitigation.md`](./docs/contamination-mitigation.md) | Preventing quality degradation in agent-generated wikis |
| [`docs/two-vault-setup.md`](./docs/two-vault-setup.md) | Obsidian two-vault setup: agent vault vs personal vault |
| [`docs/finetune-path.md`](./docs/finetune-path.md) | Turning your wiki into a fine-tuning corpus |

---

## The workflow in one diagram

```
raw/                          wiki/                        output/
───────────────────           ──────────────────────       ──────────────────
articles/          →  LLM  →  _index.md                →  reports/
papers/            compile    _concepts.md              →  slides/  (Marp)
repos/                        _graph.md                 →  figures/ (matplotlib)
datasets/                     concepts/*.md
images/                       summaries/*.md       ↑
                              topics/*.md          └── filed back into wiki
                                        ↓
                              learning/
                              _review.md           ← spaced repetition queue
                              flashcards/*.md      ← LLM-generated Q&A
                              gaps.md              ← open questions
```

---

## Quickstart

### 1. Clone and initialise your vault

```bash
git clone https://github.com/arturseo-geo/llm-knowledge-base
cd llm-knowledge-base
cp -r templates/ my-wiki/
```

Or just copy `AGENTS.md` into the root of an existing Obsidian vault.

### 2. Drop source material into `raw/`

Anything works: PDFs, markdown files clipped with Obsidian Web Clipper, `.py` scripts, `.csv` datasets, images. The LLM handles all formats.

### 3. Point your LLM at the repo and say:

```
Read AGENTS.md, then compile everything in raw/ into the wiki.
```

That's it. The agent reads the schema, creates the index files, writes concept articles, summaries, backlinks, and flashcards. From that point on, incremental ingest is a single instruction:

```
File this to our wiki: raw/papers/new-paper.pdf
```

### 4. Query against the wiki

```
What are the key differences between RLHF and Constitutional AI?
Write the answer as a report and save to output/reports/.
```

### 5. Review your learning queue

```
Show me what's due for review in learning/_review.md and run a flashcard session.
```

---

## The learning layer

The gap in every existing LLM knowledge base workflow: collecting and organising knowledge is not the same as mastering it.

This schema adds a structured learning layer under `learning/`:

- **Flashcards** are generated automatically whenever a concept article is created or updated
- **A review queue** tracks due dates using a simplified FSRS spaced repetition algorithm
- **Gap tracking** captures open questions the LLM detects during linting — turning weak spots in your wiki into a research agenda
- **Socratic Q&A** — ask the LLM to quiz you on any concept, and it grades answers against the wiki, not against general knowledge

The result: your wiki compounds twice. Once as a knowledge store, and again as a learning system that actively surfaces what you don't yet know.

---

## AGENTS.md — the schema spec

`AGENTS.md` is the contract between you and any LLM operating on your knowledge base. It defines:

- Directory layout and file naming conventions
- Frontmatter schema for every wiki article
- How the three index files (`_index.md`, `_concepts.md`, `_graph.md`) are maintained
- Step-by-step compilation, query, and linting workflows
- Learning layer: flashcard format, scheduling rules, gap tracking
- Quality rules: no hallucinated citations, confidence levels, orphan prevention
- Contamination mitigation: sandbox-first promotion model
- Session startup checklist

The spec is versioned. Breaking changes increment the major version. The agent checks the version header on every session and adapts to schema changes immediately.

[Read the full spec →](./AGENTS.md)

---

## Why not RAG?

For personal and team knowledge bases at typical research scale (50–1,000 articles), a compiled markdown wiki outperforms a vector RAG stack on every practical dimension:

| | Compiled wiki (this) | Vector RAG |
|---|---|---|
| **Retrieval** | Index files + LLM reasoning | Embedding similarity search |
| **Transparency** | Every answer traces to a file | Black-box chunk retrieval |
| **Editability** | Human-readable markdown | Opaque vector store |
| **Setup** | Zero infrastructure | Vector DB, embeddings pipeline |
| **Cost** | LLM tokens only | LLM + embedding API + DB hosting |
| **Sweet spot** | 50–2,000 articles | 10,000+ documents |

At scale (millions of documents, heterogeneous corpora, sub-second latency requirements), RAG is the right call. For a focused research domain, you probably don't need a million documents — you need the right 200, well-compiled and well-linked.

---

## Recommended tools

**IDE:** [Obsidian](https://obsidian.md) — local-first markdown vault with graph view, backlinks, and a plugin ecosystem. Free for personal use.

**Ingest:** [Obsidian Web Clipper](https://obsidian.md/clipper) — clips web articles to `.md` directly into your vault.

**LLM:** Claude (Sonnet for compilation and Q&A, Haiku for cheap linting sweeps), or any model with a large context window and good instruction-following.

**Slides:** [Marp](https://marp.app) — render Marp-format markdown as presentation slides, viewable in Obsidian.

**Search:** Start with ripgrep over the wiki directory. Upgrade to a local vector index (ChromaDB + Nomic Embed via Ollama) only when you exceed ~1,000 articles.

**Version control:** Git. Your wiki is just files — version it like code.

---

## Contamination mitigation

Agent-generated content carries quality risk — and in Obsidian, the risk goes beyond content quality. Obsidian's search, graph, backlinks, and quick switcher all operate at the vault level. Once agent-compiled articles mix with your personal notes, Obsidian stops being a representation of your knowledge and becomes a representation of the LLM's synthesis.

The primary solution is the **two-vault model**: one Obsidian vault for the agent (full schema), one for your personal notes (`insights/` — human-written only, agent never touches it). See [`docs/two-vault-setup.md`](./docs/two-vault-setup.md) for the full setup guide.

For single-repo setups, the schema enforces a sandbox-first model:

- The LLM writes freely to `output/` and `learning/`
- Promotion to `wiki/` requires either explicit instruction or passing the quality rules in `AGENTS.md §11`
- The agent **never writes to `insights/`** — this directory is human-only by schema rule
- Articles with `confidence: speculative` are not linked into the graph until upgraded
- The linting workflow quarantines contradictory articles with a `status: quarantined` flag

The distinction Steph Ango (@kepano) put precisely: a summary of a PDF is noise. An insight you formed from reading it is signal. The schema keeps them in separate directories with separate authorship rules.

---

## Finetune path

As your wiki grows and is repeatedly linted, it becomes a progressively cleaner, deduplicated, consistently-styled representation of your research domain. At that point it is more than a context store — it is a candidate training corpus.

The `learning/flashcards/` directory generates structured Q&A pairs automatically. Combined with the concept articles and summaries, this gives you the raw material for supervised fine-tuning of a smaller, domain-specialised model.

Tools for this path: [Distilabel](https://github.com/argilla-io/distilabel) for synthetic data generation, [Unsloth](https://github.com/unslothai/unsloth) or [Axolotl](https://github.com/OpenAccess-AI-Collective/axolotl) for LoRA fine-tuning.

[Full finetune guide →](./docs/finetune-path.md)

---

## Contributing

The schema is intentionally opinionated. If you use it and find patterns that work better, open an issue or PR. The goal is a community-maintained standard, not a personal config file.

Areas where contributions are most welcome:
- Additional output format templates (reveal.js, Jupyter notebooks)
- Alternative scheduling algorithms for the learning layer
- Worked example wikis on different research domains
- CLI tooling that automates the linting workflow

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## Schema version

Current: **1.1.0** — see [CHANGELOG.md](./CHANGELOG.md) for full history.

---

## Related

- [Andrej Karpathy's original post](https://x.com/karpathy/status/2039805659525644595) — the workflow that inspired this
- [Karpathy's follow-up gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — full architecture, philosophy, and tooling
- [Steph Ango on contamination and vault separation](https://x.com/kepano/status/2039831289533227446) — the two-vault model and insight vs noise distinction, by Obsidian's co-creator
- [Obsidian Spaced Repetition plugin](https://github.com/st3v3nmw/obsidian-spaced-repetition) — manual flashcards in Obsidian; this schema automates generation
- [Project Second Brain](https://layerbylayer.ai/posts/2026_02_11_project_second_brain/) — a more complex implementation with Neo4j and a web app
- [awesome-llm-knowledge-bases](https://github.com/SingggggYee/awesome-llm-knowledge-bases) — curated tool list for the ecosystem

---

## Attributions & Licence

Built and maintained by **[Artur Ferreira](https://github.com/arturseo-geo)** · Part of the **[The GEO Lab](https://thegeolab.net)** toolkit.

MIT — see [LICENSE](LICENSE). Copy freely, adapt for your domain, keep the schema versioned.

---

Found this useful? ⭐ Star the repo and connect:
[🌐 thegeolab.net](https://thegeolab.net) · [𝕏 @TheGEO_Lab](https://x.com/TheGEO_Lab) · [LinkedIn](https://linkedin.com/in/arturgeo) · [Reddit](https://www.reddit.com/user/Alternative_Teach_74/)

## Related Repos

- [claude-code-skills](https://github.com/arturseo-geo/claude-code-skills) — All 17 skills in one collection
- [seo-geo-skill](https://github.com/arturseo-geo/seo-geo-skill) — SEO & GEO optimisation skill
- [grounded-research-skill](https://github.com/arturseo-geo/grounded-research-skill) — Anti-hallucination research mode
- [content-pipeline-skill](https://github.com/arturseo-geo/content-pipeline-skill) — 7-agent content pipeline with GEO Quality Gate
