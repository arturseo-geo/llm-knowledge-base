# AGENTS.md — LLM Knowledge Base Schema
> Version: 1.1.0 | Status: Stable | Last updated: 2026-04-06

This file is the **single source of truth** for any LLM agent operating on this knowledge base. It defines the directory structure, file conventions, operational rules, and quality standards the agent must follow. The agent reads this file at the start of every session.

---

## 1. Repository layout

```
/
├── AGENTS.md          ← this file (agent reads first, always)
├── raw/               ← source material, never edited by agent
│   ├── articles/
│   ├── papers/
│   ├── repos/
│   ├── datasets/
│   └── images/
├── wiki/              ← LLM-compiled knowledge base (agent owns this)
│   ├── _index.md      ← master index: one line per article, always kept current
│   ├── _concepts.md   ← flat list of all named concepts with one-line definitions
│   ├── _graph.md      ← adjacency list of concept→concept links
│   ├── concepts/      ← one .md per named concept
│   ├── summaries/     ← one .md per raw/ source document
│   └── topics/        ← topic-level overview articles (cross-concept)
├── insights/          ← human-written notes only. Agent never writes here.
│   └── *.md           ← your own thinking: observations, connections, questions
├── output/            ← query results, slides, charts (agent writes, human reads)
│   ├── reports/
│   ├── slides/        ← Marp .md files
│   └── figures/       ← matplotlib .png files
└── learning/          ← structured learning layer (agent writes, human reviews)
    ├── _review.md     ← spaced repetition queue: concept, due date, interval, ease
    ├── flashcards/    ← one .md per concept with Q&A pairs
    └── gaps.md        ← detected knowledge gaps and open questions
```

---

## 2. Agent identity and scope

The agent is the **sole author and maintainer** of everything under `wiki/`, `output/`, and `learning/`. The human never edits these directories directly.

The agent **never modifies** anything under `raw/` or `insights/`. Raw files are immutable source inputs. Insights files are immutable human outputs — your own thinking, not the agent's synthesis.

The distinction matters: `wiki/` contains what the LLM compiled. `insights/` contains what you actually thought. A summary of a source is noise. An insight you formed from reading it is signal. These belong in different directories with different authorship rules.

The agent **always reads** `AGENTS.md` before any operation. If the schema has changed since the last session, it adapts immediately.

---

## 3. File naming conventions

| Directory | Pattern | Example |
|---|---|---|
| `wiki/concepts/` | `kebab-case.md` | `transformer-attention.md` |
| `wiki/summaries/` | mirrors `raw/` filename | `raw/papers/attention-is-all-you-need.pdf` → `wiki/summaries/attention-is-all-you-need.md` |
| `wiki/topics/` | `kebab-case.md` | `sequence-modelling.md` |
| `output/reports/` | `YYYY-MM-DD-slug.md` | `2026-04-05-attention-survey.md` |
| `output/slides/` | `YYYY-MM-DD-slug.md` | `2026-04-05-weekly-review.md` |
| `output/figures/` | `YYYY-MM-DD-slug.png` | `2026-04-05-loss-curve.png` |
| `learning/flashcards/` | mirrors `wiki/concepts/` | `transformer-attention.md` |

All filenames: lowercase, hyphens only, no spaces, no special characters.

---

## 4. Article schema

Every file under `wiki/concepts/` and `wiki/topics/` opens with this frontmatter block:

```yaml
---
title: "Transformer Attention"
type: concept          # concept | topic | summary
created: 2026-04-05
updated: 2026-04-05
confidence: high       # high | medium | low | speculative
sources:
  - ../summaries/attention-is-all-you-need.md
  - ../summaries/bert-paper.md
related:
  - ../concepts/multi-head-attention.md
  - ../concepts/positional-encoding.md
  - ../topics/sequence-modelling.md
tags: [attention, transformer, nlp]
---
```

`confidence` reflects how well-sourced the article is. The agent downgrades confidence when sources are thin or when the linter finds contradictions.

---

## 5. Index maintenance

The agent **updates `wiki/_index.md` after every write operation**. Format:

```markdown
| Article | Type | Confidence | Updated | Tags |
|---|---|---|---|---|
| [Transformer Attention](concepts/transformer-attention.md) | concept | high | 2026-04-05 | attention, transformer |
```

The agent **updates `wiki/_concepts.md`** whenever a new concept article is created:

```markdown
- **Transformer Attention** — mechanism by which tokens attend to all other tokens with learned weights. → [article](concepts/transformer-attention.md)
```

The agent **updates `wiki/_graph.md`** when backlinks change:

```markdown
transformer-attention → multi-head-attention
transformer-attention → positional-encoding
transformer-attention → sequence-modelling (topic)
```

These three index files are the primary navigation layer. The agent reads them before deciding which full articles to load for any query.

---

## 6. Compilation workflow

When the agent receives a new raw document (e.g. `"file this to our wiki: raw/papers/new-paper.pdf"`):

1. **Read** `AGENTS.md`, `wiki/_index.md`, `wiki/_concepts.md`
2. **Summarise** the raw document → write `wiki/summaries/<filename>.md`
3. **Identify** all named concepts in the document
4. For each concept: **check** if a concept article already exists
   - If yes: update it, add the new source, revise confidence if needed
   - If no: create `wiki/concepts/<slug>.md` with full frontmatter
5. **Check** for topic-level articles that should link to this material — update if found
6. **Update** `_index.md`, `_concepts.md`, `_graph.md`
7. **Report** what was created, updated, and any new open questions flagged

---

## 7. Query workflow

When the agent receives a research question:

1. **Read** `_index.md` and `_concepts.md` to identify relevant articles
2. **Load** the relevant concept and topic articles (not the full raw/ corpus)
3. **Answer** the question with citations to specific wiki files
4. **Write** the answer to `output/reports/YYYY-MM-DD-<slug>.md`
5. **Offer** to file useful findings back into the wiki as new or updated articles

---

## 8. Linting workflow

The agent runs a health check when asked (or on a schedule). Steps:

1. **Orphan detection** — find articles with no backlinks in `_graph.md`
2. **Contradiction scan** — flag articles with conflicting claims across sources
3. **Confidence audit** — downgrade articles where sources are older than 6 months or sparse
4. **Gap detection** — identify concepts mentioned in articles but lacking their own entry
5. **Web imputation** — for low-confidence or gap articles, use web search to fill missing data, mark additions with `source: web-imputed`
6. **Connection suggestions** — propose new `_graph.md` edges and candidate topic articles
7. **Write results** to `output/reports/YYYY-MM-DD-lint.md`

---

## 9. Learning layer

The agent maintains a lightweight spaced repetition system under `learning/`.

### Flashcard format (`learning/flashcards/<concept>.md`)

```markdown
---
concept: transformer-attention
last_reviewed: 2026-04-05
interval_days: 3
ease: 2.5
---

Q: What problem does the attention mechanism solve in sequence modelling?
A: It allows each token to directly attend to any other token in the sequence, regardless of distance, solving the long-range dependency problem that RNNs struggle with.

Q: What are the three components of scaled dot-product attention?
A: Query (Q), Key (K), and Value (V) — all learned linear projections of the input.
```

### Review queue (`learning/_review.md`)

```markdown
| Concept | Due | Interval | Ease | Status |
|---|---|---|---|---|
| transformer-attention | 2026-04-08 | 3d | 2.5 | due |
| positional-encoding | 2026-04-12 | 7d | 2.3 | upcoming |
```

### Scheduling rules

The agent applies a simplified FSRS-style algorithm:
- New card: interval = 1 day
- After correct recall: interval = interval × ease (default ease = 2.5)
- After failed recall: interval = 1 day, ease -= 0.2 (floor: 1.3)
- After perfect recall: ease += 0.1 (ceiling: 3.0)

The agent generates new flashcards whenever a concept article is created or substantially updated.

### Gap tracking (`learning/gaps.md`)

```markdown
## Open questions

- [ ] How does Flash Attention differ from standard attention in memory complexity?
  - Source: mentioned in `summaries/flash-attention.md`, no concept article yet
- [ ] What is the relationship between rotary position embeddings and ALiBi?
  - Source: agent-detected during linting pass 2026-04-05
```

---

## 10. Output formats

| Request type | Output format | Location |
|---|---|---|
| Research question | Markdown report | `output/reports/` |
| Summary / briefing | Markdown | `output/reports/` |
| Presentation | Marp slide deck | `output/slides/` |
| Chart / visualisation | matplotlib PNG | `output/figures/` |
| Flashcard review session | Markdown Q&A | `output/reports/` |

All outputs include a header block:

```markdown
---
query: "What is the state of long-context transformers?"
generated: 2026-04-05
sources: [concepts/transformer-attention.md, topics/long-context.md]
filed_back: false   # set to true if the agent incorporates this output into the wiki
---
```

---

## 11. Quality rules

- **Never hallucinate citations.** Every claim in the wiki must trace to a file in `raw/` or be marked `source: web-imputed` or `source: agent-inferred`.
- **Confidence floor.** Articles with fewer than 2 sources must be marked `confidence: low`.
- **No orphans.** Every concept article must appear in at least one topic article or `_graph.md` edge.
- **Flat is fine.** Do not create subdirectories inside `wiki/concepts/` or `wiki/topics/`. The flat structure scales to 1,000+ articles without reorganisation.
- **One concept per file.** Articles that cover two distinct concepts must be split.
- **Backlinks are bidirectional.** If A links to B, B must link back to A in its frontmatter.
- **Output ≠ wiki.** Files in `output/` are query artifacts. They are not wiki articles. The agent must explicitly decide to `filed_back: true` and write the insight into the appropriate wiki file.
- **Never write to `insights/`.** This directory is reserved exclusively for human-written notes. Agent-generated content — even high-confidence, well-sourced synthesis — does not belong here. The separation is the point.

---

## 12. Contamination mitigation

The agent operates in a **sandbox-first** model:

- All generated content is written to `output/` or `learning/` by default
- Promotion to `wiki/` requires either explicit human instruction or the agent meeting the quality rules in §11
- The agent flags any article where confidence is `speculative` and does not link it into `_graph.md` until confidence is upgraded
- During linting, the agent quarantines contradictory articles by adding `status: quarantined` to frontmatter until the contradiction is resolved

---

## 13. Session startup checklist

At the start of every session, the agent confirms:

- [ ] `AGENTS.md` read and schema version noted
- [ ] `wiki/_index.md` loaded (article count: ___)
- [ ] `wiki/_concepts.md` loaded (concept count: ___)
- [ ] `learning/_review.md` checked (cards due today: ___)
- [ ] Ready to receive instructions

---

## 14. Schema versioning

This file follows semantic versioning. The agent checks the version header on every session.

| Version | Changes |
|---|---|
| 1.1.0 | Added `insights/` directory (human-written only, agent never writes here). Updated §2 and §11 to enforce the authorship boundary. Added two-vault model as primary setup recommendation. |
| 1.0.0 | Initial release. Core directory layout, compilation, query, lint, learning layer. |

When the human updates `AGENTS.md`, the agent adapts immediately and notes the version change in its session startup output.

---

*AGENTS.md is a living document. The schema evolves with the knowledge base. The agent always defers to the version in this file, not to assumptions from prior sessions.*
