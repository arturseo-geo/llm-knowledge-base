# Changelog

All schema changes are documented here. Follows [Semantic Versioning](https://semver.org/).

- **Major** version: breaking changes to directory layout or frontmatter schema
- **Minor** version: new sections, new optional fields, new workflow steps
- **Patch** version: clarifications, typo fixes, non-breaking rewording

Agents check the version header in `AGENTS.md` on every session startup and adapt immediately to any version they encounter.

---

## [1.1.0] — 2026-04-06

Prompted by Steph Ango (@kepano, co-creator of Obsidian) on the insight/noise distinction and the Obsidian-specific contamination problem.

### Added
- `insights/` directory to the schema — human-written notes only, agent never writes here
- `docs/two-vault-setup.md` — practical guide for the Obsidian two-vault model
- Two-vault model promoted to primary recommended setup

### Changed
- `AGENTS.md §1` — `insights/` added to directory layout
- `AGENTS.md §2` — agent scope updated: never modifies `raw/` or `insights/`; insight vs summary distinction explained
- `AGENTS.md §11` — new rule: agent never writes to `insights/`
- `docs/contamination-mitigation.md` — restructured around nuclear steel analogy; adds Obsidian-specific vault corruption concern; two-vault model becomes primary section
- `README.md` — contamination section rewritten; contents table and Related section updated with Kepano attribution and direct links

---

## [1.0.0] — 2026-04-05

Initial release.

### Added
- Core directory layout: `raw/`, `wiki/`, `output/`, `learning/`
- Three index files: `_index.md`, `_concepts.md`, `_graph.md`
- Article frontmatter schema with `confidence`, `sources`, `related`, `tags`
- Compilation workflow (§6): ingest → summarise → concept extraction → index update
- Query workflow (§7): index-first retrieval, report output, file-back option
- Linting workflow (§8): orphan detection, contradiction scan, gap detection, web imputation
- Learning layer (§9): flashcard format, FSRS-style scheduling, review queue, gap tracking
- Output format table (§10) with header block schema
- Quality rules (§11): citation traceability, confidence floor, orphan prevention
- Contamination mitigation (§12): sandbox-first promotion model, quarantine flag
- Session startup checklist (§13)
- Templates: `concept.md`, `summary.md`, `flashcard.md`, `topic.md`, `_index.md`, `_concepts.md`, `_graph.md`, `output-report.md`, `lint-report.md`
- Docs: `why-not-rag.md`, `learning-layer.md`, `contamination-mitigation.md`
- Example wiki: 5 concept articles on AI alignment (AI safety topic)
