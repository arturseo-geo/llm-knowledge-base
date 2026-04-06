# Contamination mitigation

In the 1940s, nuclear weapons testing contaminated nearly all steel produced after the detonations with elevated background radiation. Sensitive instruments now require "low-background steel" — metal that was smelted before 1945, often salvaged from pre-war shipwrecks. The contamination was invisible, pervasive, and irreversible in the general supply.

Steph Ango (@kepano), co-creator of Obsidian, made the same analogy for LLM-generated content: the training data contamination from synthetic text is the new low-background steel problem. Once agent-generated content mixes with human-created knowledge, it's difficult to know which ideas are yours, which are sourced, and which are confabulated.

This doc describes how contamination happens in an LLM knowledge base, why it's particularly destructive in Obsidian, and the concrete patterns this schema uses to prevent it.

---

## Why contamination is worse in Obsidian than you think

Most discussions of contamination focus on content quality — wrong facts, circular citations, hallucinated sources. Those are real. But Steph Ango identified a subtler problem specific to Obsidian: contamination corrupts the *tool itself*, not just the content.

Obsidian's search, graph view, backlinks, quick switcher, and Bases all operate at the vault level. They don't distinguish between human-written notes and agent-compiled articles. Once agent content enters your personal vault, every search result, every graph node, every backlink suggestion is polluted with content that may not represent your thinking at all.

The result: Obsidian stops being a representation of your knowledge and becomes a representation of your LLM's synthesis. Those are different things. Search is no longer scoped to your knowledge. The graph no longer maps your thinking. Backlinks surface connections the agent made, not connections you made.

This is why the two-vault model is the **primary recommended setup**, not an optional enhancement.

---

## The two-vault model (primary setup)

Keep two completely separate Obsidian vaults:

**Agent vault** — the LLM's working environment. Contains the full schema: `raw/`, `wiki/`, `output/`, `learning/`. The agent reads and writes here freely. This is where compilation, linting, Q&A, and flashcard generation happen. Your Obsidian search, graph, and backlinks in this vault reflect the compiled knowledge base.

**Personal vault** — your knowledge, scoped to your thoughts. Contains only `insights/` — notes you wrote yourself. Nothing agent-generated enters here unless you deliberately copy a specific artifact you've validated and made your own. Your Obsidian search, graph, and backlinks in this vault reflect only your thinking.

Promotion from the agent vault to the personal vault is the highest-value action in the workflow. It happens rarely and deliberately: only when a compiled article, flashcard session, or query output has produced something you genuinely understand and want to own. A summary of a PDF is noise. An insight you formed from engaging with the compiled wiki is signal.

To implement: create two separate Obsidian vaults on your filesystem. Point the agent at the agent vault. Keep the personal vault entirely human-written.

---

## The single-repo model (simpler alternative)

If you're not using Obsidian, or prefer a single repository, the schema's `insights/` directory provides a lighter version of the same separation.

The rule is absolute: **the agent never writes to `insights/`**. This directory is human-only. Everything else — `wiki/`, `output/`, `learning/` — is agent territory.

The tradeoff: in Obsidian, your search and graph will still include agent-compiled content alongside your insights. Use Obsidian's "Excluded files" setting to exclude `wiki/`, `output/`, and `learning/` from search and graph if you want cleaner scoping. The two-vault model is cleaner.

---

## Confidence levels

Every wiki article carries a `confidence` field in its frontmatter:

```yaml
confidence: high       # high | medium | low | speculative
```

The agent assigns confidence based on sourcing:

| Level | Criteria |
|---|---|
| `high` | 2+ primary sources, no contradictions detected, recently linted |
| `medium` | 1–2 sources, or sources older than 6 months, or minor ambiguities |
| `low` | Single source, or thin coverage, or imputed from web search |
| `speculative` | Agent-inferred, no direct source, or contradictions unresolved |

**Speculative articles are not linked into the graph.** The `_graph.md` adjacency list only includes articles with `confidence: medium` or above. This prevents speculative content from propagating through backlinks into otherwise well-sourced articles.

The linting workflow audits confidence levels and downgrades articles where:
- Sources have become stale (>6 months old for fast-moving domains)
- The article has fewer than 2 sources
- A contradiction has been detected

Confidence can only be **upgraded** when new sources are added or contradictions resolved — never automatically.

---

## Quarantine

When a contradiction is detected between two articles or between an article and a source, the linting workflow adds a quarantine flag:

```yaml
status: quarantined
quarantine_reason: "Contradicts claims in summaries/source-b.md re: claim X"
quarantine_date: YYYY-MM-DD
```

Quarantined articles:
- Are excluded from `_graph.md` until the contradiction is resolved
- Are marked visibly in `_index.md` with a `[quarantined]` tag
- Are not used as sources for new article synthesis
- Appear as the first items in the next linting report

Resolution options:
1. The human reviews both sources and decides which is correct → agent updates the article and removes the flag
2. The contradiction is real and both claims are valid in different contexts → agent splits the article or adds a nuance section
3. One source supersedes the other → agent updates confidence and provenance

---

## Citation traceability

Every claim in a wiki article must trace to a file in `raw/` or be explicitly marked:

```yaml
source: web-imputed     # filled in by agent using web search during linting
source: agent-inferred  # logical inference from other wiki articles, no direct source
```

The agent never writes claims without one of:
- A citation to a `summaries/` file (which traces back to `raw/`)
- A `source: web-imputed` marker with a note on the search query used
- A `source: agent-inferred` marker with the reasoning chain noted

This makes every article auditable. When you read a claim and want to verify it, you can follow the citation chain from the wiki article back to the original raw source.

---

## Output file-back discipline

The most common contamination path is this: you run a query, get a useful answer, and ask the agent to file it back into the wiki. The answer was generated by the LLM reasoning over wiki articles — it's not a new source, it's a synthesis. If it gets filed as an article with `confidence: high`, you've created a circular citation: the wiki citing itself.

The schema prevents this through the `filed_back` and `filed_back_to` fields in output reports:

```yaml
filed_back: true
filed_back_to:
  - concepts/concept-a.md   # added one paragraph under "Further context"
  - topics/topic-b.md       # added to open questions section
```

When a query output is filed back, the agent:
1. Marks the source as `source: agent-inferred` in the updated wiki article
2. Sets `confidence` to no higher than `medium` regardless of the agent's confidence in the answer
3. Adds a note: "This section synthesised from query output YYYY-MM-DD-slug.md — not directly sourced"

Agent-inferred content is useful. It becomes contamination only when it's treated as equivalent to sourced content.

---

## Linting as a control loop

Contamination mitigation is not a one-time setup — it's an ongoing control loop. The linting workflow (AGENTS.md §8) is the mechanism that catches drift before it compounds:

- **Contradiction scan** catches conflicting claims that slipped through during compilation
- **Confidence audit** catches articles that looked well-sourced initially but whose sources have become stale
- **Orphan detection** catches articles that got disconnected from the graph, often a sign of a botched update
- **Gap detection** catches claims that reference concepts that were never properly sourced

Run linting regularly — at minimum, every time the wiki doubles in size. For active research, weekly linting passes are practical.

---

## Summary: the contamination controls

| Control | What it prevents |
|---|---|
| Two-vault model (primary) | Agent content ever entering your personal vault or corrupting Obsidian's search/graph |
| `insights/` directory (agent-never-writes) | Agent synthesis mixing with human thinking in a single repo |
| Sandbox-first promotion | Unreviewed query artifacts entering the wiki |
| Confidence levels | Thin or speculative content being treated as authoritative |
| Speculative articles excluded from graph | Speculation propagating through backlinks |
| Quarantine flag | Contradictions spreading into new articles |
| Citation traceability | Unsourced claims |
| Filed-back discipline | Circular citations (wiki citing itself) |
| Regular linting | All of the above, on a recurring basis |

None of these controls are perfect in isolation. Together, they make contamination visible and reversible — which is the realistic goal. A compiled wiki will contain errors. The schema is designed so that when errors are found, they can be traced, quarantined, and corrected without corrupting everything that cited them.

---

*The two-vault model and the insight/noise distinction were articulated by Steph Ango (@kepano), co-creator of Obsidian, in response to Andrej Karpathy's April 2026 LLM Knowledge Bases post.*

*For the full quality rules spec, see `AGENTS.md §11`.*
*For the linting workflow, see `AGENTS.md §8`.*
*For the sandbox promotion model, see `AGENTS.md §12`.*
*For a practical Obsidian two-vault setup guide, see `docs/two-vault-setup.md`.*
