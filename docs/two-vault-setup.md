# Two-vault setup

The two-vault model is the primary recommended setup for using this schema with Obsidian. This guide covers how to configure it, what goes where, and when to promote content between vaults.

The core principle, articulated by Steph Ango (@kepano), co-creator of Obsidian: keep your personal vault high signal-to-noise, with all content of known origin. Let agents operate in a separate vault. Only once the agent-facing workflow produces something genuinely useful — something you've understood, validated, and made your own — bring it into the personal vault.

---

## Why two vaults, not one

Obsidian's features operate at the vault level. Search, graph view, backlinks, quick switcher, Bases — all of them index every file in the vault without distinguishing between human-written notes and agent-compiled articles.

In a single-vault setup, your Obsidian experience reflects a mixture of your thinking and the LLM's synthesis. The graph maps connections the agent made. Search surfaces articles the agent wrote. Backlinks include links the agent created. Over time, Obsidian stops being a tool for thinking and becomes a tool for browsing the LLM's output.

The two-vault model keeps these separate by design, not by configuration.

---

## Filesystem structure

```
~/vaults/
├── agent-vault/          ← LLM works here
│   ├── AGENTS.md
│   ├── raw/
│   ├── wiki/
│   ├── output/
│   └── learning/
│
└── personal-vault/       ← you work here
    └── insights/
        ├── daily/
        ├── projects/
        └── topics/
```

These are two completely independent Obsidian vaults. Open them separately. The agent has no access to `personal-vault/`. You rarely open `agent-vault/` directly — you use it primarily via CLI or Claude Code sessions.

---

## Setting up the agent vault

This is the standard schema setup. Clone the repo or copy `AGENTS.md` and the templates into a new directory:

```bash
git clone https://github.com/arturseo-geo/llm-knowledge-base agent-vault
cd agent-vault
```

Open in Obsidian: **Obsidian → Open folder as vault → select `agent-vault/`**

Recommended Obsidian settings for the agent vault:
- **Files & Links → Default location for new notes:** `wiki/concepts/` — so any note you create manually lands in the right place
- **Files & Links → Excluded files:** none — you want full visibility of everything the agent creates here

---

## Setting up the personal vault

Your personal vault is just a folder of markdown files you write yourself. Structure it however suits your thinking — there's no schema requirement here. A simple starting structure:

```
personal-vault/
└── insights/
    ├── _index.md          ← your own index, manually maintained
    ├── daily/             ← daily notes, fleeting thoughts
    ├── projects/          ← project-specific notes
    └── topics/            ← topic notes that have reached some stability
```

Open in Obsidian: **Obsidian → Open folder as vault → select `personal-vault/`**

Recommended Obsidian settings for the personal vault:
- **Files & Links → Default location for new notes:** `insights/` or `insights/daily/`
- The graph, search, backlinks, and quick switcher here reflect only your notes — this is the point

---

## Workflow: how the two vaults interact

**Day-to-day:**

1. Drop new source material into `agent-vault/raw/`
2. Run a Claude session against `agent-vault/` — compile, query, lint
3. Browse compiled articles in `agent-vault/` via Obsidian if you want to explore the wiki
4. Open `personal-vault/` when you want to write your own notes

**Promotion (the high-value action):**

Promotion is the deliberate act of taking something from the agent vault and making it yours. It happens when:

- A compiled article surfaces a connection you hadn't seen and want to explore further
- A flashcard session reveals something you now genuinely understand and want to record
- A query output produces an answer that changes how you think about something
- Linting identifies a gap that you now want to write about yourself

When this happens, you write a note in `personal-vault/insights/` in your own words. You do not copy the agent's article. You write what *you* now think, drawing on what the compiled wiki helped you understand. The wiki compiled. You mastered.

**What never gets promoted:**

- LLM summaries of papers (summaries are noise — your insight from the paper is signal)
- Agent-compiled concept articles verbatim
- Output reports copied wholesale
- Anything you haven't read, understood, and processed yourself

---

## Using Obsidian's excluded files as a lighter alternative

If two vaults feels like too much friction, a single-repo setup with careful exclusions is a workable middle ground.

In `personal-vault/.obsidian/app.json` (or via Settings → Files & Links → Excluded files), add:

```
wiki/
output/
learning/
```

This excludes agent-compiled content from search, graph, and quick switcher — approximating the two-vault separation without the overhead. The tradeoff: the files are still in the same vault, so backlinks can still surface agent-created connections. The two-vault model is cleaner.

---

## Promotion queue

When the agent identifies something worth promoting, it appends to `output/promotion-queue.md`:

```markdown
## 2026-04-06

- [ ] `wiki/concepts/rlhf.md` — the section on reward model overoptimisation is worth 
  understanding properly. Consider writing an insight note on why KL-divergence is 
  the right regulariser here.
- [ ] Query output `output/reports/2026-04-06-attention-survey.md` — conclusion paragraph 
  is worth your own note. The attention → meaning connection you found is non-obvious.
```

This gives you a curated list of agent-flagged candidates for promotion. You decide which ones to actually engage with and write up in your personal vault. Most won't make it — that's fine. The ones that do are the ones that compound.

---

## The signal/noise test

Before writing anything in `personal-vault/insights/`, ask: is this a summary of what the LLM told me, or is it something I now think?

A summary of a paper → agent vault, `wiki/summaries/`
An insight I formed from reading the compiled summary → personal vault, `insights/`

The test is not about quality. A well-written agent summary is still noise if it's not yet yours. A rough, incomplete observation you wrote yourself is signal — it's the beginning of understanding.

---

*For the contamination risks of mixing agent and personal content, see `docs/contamination-mitigation.md`.*
*For the AGENTS.md rule prohibiting agent writes to `insights/`, see `AGENTS.md §2` and `§11`.*
