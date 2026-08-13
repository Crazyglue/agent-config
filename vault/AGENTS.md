# Vault — Agent Guide

This is the AI's persistent memory for all work. AI sessions have **no memory across conversations** — this vault is the memory.

The vault is a **compounding artifact**: every session must leave it richer. Sessions are conversational; the wiki is reference; raw is source-of-truth input. The wiki is what compounds.

---

## Auto-recall (do this without being asked)

**At the start of every turn**, before responding:

1. `Read` `wiki/master-index.md`. (Cheap. Always.)
2. Scan the user's message for any topic name, file path, concept, or tag listed in master-index. For each match, `Read` that topic's `_overview.md`.
3. Scan `sessions/` filenames (`ls sessions/`) for date or topic relevance. Read at most the 1–2 most relevant memos.
4. Honor every "Notes for the Next AI" section you read.

**Do not** wait for the user to say "recall from memory," "check the vault," or similar. Recall is the default; ignoring the vault is the exception. The only turns where recall is skippable are trivial (single-fact, no-context) — and even then, master-index is cheap enough to read.

If recall surfaces relevant prior work, briefly tell the user (one line, e.g., "Per the api-models-package wiki entry, the workspace extraction is already done."). Don't dump the contents — cite and proceed.

---

## Vault structure

```
vault/
├── AGENTS.md          ← this file
├── raw/               ← unprocessed source material (only on user request)
├── sessions/          ← per-conversation memos (auto-written)
└── wiki/              ← curated, compounding knowledge base
    ├── master-index.md
    └── <topic>/
        ├── _overview.md
        └── <detail>.md
```

### `raw/` — Source material

Drop zone for PRDs, ADRs, transcripts, exports. **Read only on explicit request. Never write unless asked to stage.** Process raw → wiki entry → leave raw alone unless told to delete.

### `sessions/` — Conversation memos (auto-written)

One file per substantive conversation. Naming: `YYYY-MM-DD-<short-topic>.md`.

**Required front-matter:**
```yaml
---
date: YYYY-MM-DD
topic: short description
status: active-design | resolved | archived | exploratory
participants: [user-handle, ai-orchestrator]
tags: [tag1, tag2]
related-docs:
  - path/to/doc
wiki-delta:                          # NEW — see "Eager distillation" below
  - "wiki/<topic>/<file>.md (created|updated): one-line reason"
---
```

**Required sections in this order:**
1. **Why this memo exists** — one paragraph; tells the future reader whether to read in full or skip to TL;DR.
2. **TL;DR / Current Recommendation** — bottom line, standalone-readable.
3. Body — context, options, decisions, evidence, references.
4. **How the conversation evolved** — dead-end paths and reframings, so future sessions don't repeat them.
5. **Notes for the Next AI / Future-Me** — explicit "don't propose X / always check Y" guidance.
6. **Wiki delta** — list of wiki pages this session created or updated and a one-line "what changed" per page. Mirrors front-matter `wiki-delta` but in narrative form. **Required even if empty** (state "no wiki changes — purely tactical session").

**Auto-record triggers** (any of, write/update memo without asking):
- Architectural decision or recommendation
- Non-trivial design discussion (multiple options weighed)
- Codebase reality discovery (vs. how docs say it works)
- Mid-session reframing/reversal
- Any "Notes for the Next AI" insight worth preserving

**Do not record:** trivial single-fact lookups, pure execution with no decisions, conversational chatter.

### `wiki/` — Curated knowledge base (the compounding layer)

The durable source of truth. Reference docs, not transcripts.

#### `wiki/master-index.md`

Catalog of every topic with one-line description and link to `_overview.md`. **Always read first.** Update whenever a topic is added.

#### `wiki/<topic>/_overview.md`

Entry point for the topic. Required structure:
- **What this topic covers** — one paragraph
- **Key concepts** — short glossary if useful
- **Documents in this topic** — link every other file with one-sentence description
- **Related topics** — wikilinks to other `_overview.md`
- **Related sessions** — wikilinks to relevant `sessions/*.md`
- **Related raw docs** — wikilinks to `raw/*` if applicable

#### `wiki/<topic>/<detail>.md`

Reference docs. Each must be linked from `_overview.md`. **If it's not linked, it doesn't exist.**

---

## Eager distillation (the core operation)

Every qualifying session **must** end with a wiki update. This is non-negotiable — it's how the compounding artifact accumulates value instead of stagnating in session memos.

### Promotion rules

| Session content | Promote to wiki? |
|---|---|
| Architectural decision with rationale | **Yes** — extend or create topic page |
| Codebase discovery contradicting docs | **Yes** — note in topic detail page |
| Reusable gotcha / debugging pattern | **Yes** — gotcha page under relevant topic |
| Pure execution log (commits made, files touched) | **No** — stays in session memo only |
| Speculative/exploratory dead end | **Maybe** — only if the dead-end itself is reusable knowledge ("don't try X because Y") |
| Q&A query result that synthesizes ≥2 prior topics | **Yes** — file as new wiki page (see "File-back" below) |

### Distillation workflow (run before closing any qualifying session)

1. Identify which existing wiki topic(s) this session affects. If none, propose a new topic to the user.
2. For each affected topic:
   - Update or create the relevant detail page(s).
   - Update the topic's `_overview.md` (link new files; revise descriptions if scope changed).
3. If a new topic was created, update `master-index.md`.
4. Record what changed in the session memo's `wiki-delta` section and front-matter.
5. Distillation is **distillation**, not transcript copy-paste. Wiki entries should read like reference docs — bullet lists, tables, short prose. Strip narrative.

### File-back: query results become wiki pages

When the user asks an analytical or comparative question and the answer is non-trivial, **the answer itself is a wiki candidate.** Examples:
- "How does X compare to Y?" → comparison page under one of the topics
- "What are all the places we do Z?" → reference page enumerating
- "Why did we choose A over B?" → ADR-style page

Default behavior: if your answer would be useful to a future session asking the same question, file it. Tell the user briefly ("Filing this as `wiki/<topic>/<file>.md`").

---

## Lint operation (run on request, or proactively at session start when scope is broad)

The lint pass keeps the wiki healthy as it grows. Run it when:
- The user asks ("lint the vault", "check vault health")
- You notice symptoms (sessions overlap heavily, master-index feels stale, new topic borders an existing one)
- Quarterly housekeeping

### Lint checklist

1. **Orphan sessions** — sessions with substantive content but no `wiki-delta` and no corresponding wiki entry. List them; propose distillation targets.
2. **Overlapping sessions** — two+ sessions on the same topic where the wiki has only fragmentary coverage. Propose consolidation into a single topic page.
3. **Orphan wiki pages** — files in `wiki/<topic>/` not linked from `_overview.md`. Either link them or delete.
4. **Stale claims** — wiki pages whose source sessions are old AND whose codebase references may have moved. Spot-check 1–2 file paths per page.
5. **Missing topics** — concepts mentioned in 3+ sessions with no dedicated wiki topic. Propose new topic.
6. **Master-index drift** — topics in `wiki/` not listed in `master-index.md`, or descriptions out of date.
7. **Cross-reference gaps** — topic A references topic B in prose but no wikilink. Add the wikilink.
8. **Contradictions** — same claim in two pages with different conclusions. Flag for user resolution; don't silently pick a winner.

Output a short report: `Lint findings: N orphans, M overlaps, K stale, …` then propose actions ranked by leverage.

---

## Workflow patterns

### Pattern: Resume a topic in a new conversation
1. Auto-recall (master-index → relevant `_overview.md` → relevant session memos).
2. Honor "Notes for the Next AI" sections.
3. Re-hydrate before recommending anything.

### Pattern: Process a raw document into the wiki
1. Read raw doc.
2. Identify target topic (or propose new).
3. Create/extend wiki entry; update topic `_overview.md`; update `master-index.md` if new topic.
4. Confirm with user before deleting the raw doc.

### Pattern: Record a session (auto)
1. Create memo file early in the conversation (`sessions/YYYY-MM-DD-<topic>.md`); update incrementally.
2. Before ending: complete required sections; run distillation; record `wiki-delta`.
3. Tell the user briefly that you're recording / what wiki pages you updated. Don't ask permission.

### Pattern: Lint
1. Walk the lint checklist.
2. Output ranked findings.
3. Execute fixes the user approves; record as a session memo with `wiki-delta`.

---

## Conventions

- **Wikilinks (`[[...]]`)** preferred over relative paths.
- **Front-matter required** on session memos and every `_overview.md`. Optional but encouraged on detail pages.
- **Tags consistent** — reuse existing tags from master-index/other files before inventing new ones.
- **Date format:** `YYYY-MM-DD`.
- **Markdown only.** No exotic Obsidian plugins assumed.
- **Prefer tables, bullet lists, and short sections** over long paragraphs — faster AI re-hydration.
- **Always update master-index** when topics change.

---

## What NOT to do

- Don't skip auto-recall. The vault is the memory; ignoring it defeats the system.
- Don't end a qualifying session without a wiki delta (even if empty, state it explicitly).
- Don't dump session transcripts into the wiki — distill.
- Don't write to `raw/` without being asked.
- Don't create wiki topics without an `_overview.md`.
- Don't add files to a topic subfolder without linking from `_overview.md`.
- Don't ask permission to record a session — just do it and announce briefly.
- Don't silently resolve contradictions found during lint — flag for the user.
