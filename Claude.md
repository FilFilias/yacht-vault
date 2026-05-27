# YachtBay Knowledge Vault

A structured, interlinked knowledge base for building YachtBay — a peer-to-peer yacht rental marketplace.
Claude maintains the wiki. The human curates sources, makes decisions, and guides the build.

Based on Andrej Karpathy's LLM Wiki pattern, extended for product development.

---

## What is YachtBay

YachtBay is a P2P yacht rental marketplace launching in Greece. Yacht owners list their vessels; renters browse, compare, and instantly book. The platform supports day trips and multi-day/week charters, with three crew configurations: bareboat, skippered, and fully crewed. Revenue model: commission from owners (~13%, flexible per owner). Target customer: mid-range travelers.

---

## Vault Structure

```
raw/                    -- immutable source documents (never modify)
  competitors/          -- competitor research, screenshots, notes
  market-research/      -- market data, reports, articles
  regulations/          -- Greek maritime law, licensing requirements

wiki/                   -- Claude-maintained knowledge pages
  index.md              -- master table of contents (always kept up to date)
  hot.md                -- session cache: recent decisions, current focus, open questions
  log.md                -- append-only record of all operations
  concepts/             -- business and product concepts (business model, commission, personas, etc.)
  entities/             -- competitors, partners, key people
  sources/              -- summaries of ingested raw documents

decisions/              -- one file per key decision (ADR format)
specs/                  -- product and technical specifications
roadmap/                -- phases, milestones, MVP scope
```

---

## Session Start Protocol

At the start of every session:
1. Read `wiki/hot.md` — get context on recent decisions and current focus
2. If the user's question requires broader context, read `wiki/index.md`
3. Only then drill into specific wiki, spec, or decision pages as needed

---

## Workflows

### Ingest (external source → wiki)

When the user adds a document to `raw/` and asks you to ingest it:

1. Read the full source document
2. Discuss key takeaways with the user before writing anything
3. Create a summary page in `wiki/sources/` named after the source
4. Create or update concept/entity pages for each major idea
5. Add `[[wikilinks]]` to connect related pages
6. Update `wiki/index.md` with new pages and one-line descriptions
7. Update `wiki/hot.md` with what changed and why it matters
8. Append an entry to `wiki/log.md` with the date, source name, and what changed

A single source may touch 10–15 wiki pages. That is normal.

### Draft (new product doc → specs or wiki)

When the user asks you to document a feature, spec, or concept:

1. Discuss the scope and key points with the user first
2. Create the appropriate file (`specs/`, `wiki/concepts/`, or `roadmap/`)
3. Link to related decisions (`[[decisions/...]]`) and wiki pages
4. Update `wiki/index.md` and `wiki/hot.md`
5. Append an entry to `wiki/log.md`

### Decide (key decision → decisions/)

When the user makes or discusses a significant decision (architecture, business model, product, design):

1. Create a new file in `decisions/` using the ADR format below
2. Link from the relevant wiki or spec pages
3. Update `wiki/hot.md` with the decision summary
4. Append an entry to `wiki/log.md`

### Query (question → answer)

When the user asks a question:

1. Read `wiki/hot.md` first
2. Read `wiki/index.md` to find relevant pages
3. Read those pages and synthesize an answer
4. Cite specific pages in your response using `[[wikilinks]]`
5. If the answer is not in the vault, say so clearly
6. If the answer is valuable, offer to save it as a new wiki page

Good answers should be filed back into the wiki so they compound over time.

---

## Page Formats

### Wiki page (`wiki/concepts/`, `wiki/entities/`, `wiki/sources/`)

```markdown
---
title: Page Title
tags:
  - concept | entity | source
last-updated: YYYY-MM-DD
---

# Page Title

**Summary**: One to two sentences describing this page.

**Sources**: [[source-file]] (for concept/entity pages that draw from raw docs)

---

Main content. Use clear headings and short paragraphs.
Link to related concepts using [[wikilinks]] throughout.

## Related Pages

- [[related-page-1]]
- [[related-page-2]]
```

### Decision record (`decisions/`)

File name: `YYYY-MM-DD-short-title.md`

```markdown
---
title: Decision Title
date: YYYY-MM-DD
status: proposed | accepted | superseded
tags:
  - decision
---

# Decision Title

## Context

What situation or problem prompted this decision?

## Options Considered

- **Option A**: description — pros / cons
- **Option B**: description — pros / cons

## Decision

What was decided and why.

## Consequences

What this means going forward. What becomes easier or harder.

## Related

- [[related-wiki-page]]
- [[related-spec]]
```

### Spec page (`specs/`)

```markdown
---
title: Spec Title
tags:
  - spec
status: draft | review | approved
last-updated: YYYY-MM-DD
---

# Spec Title

**Summary**: What this spec covers.

---

## Overview

## Requirements

## User Flows

## Edge Cases & Rules

## Open Questions

## Related

- [[decision]]
- [[wiki-concept]]
```

---

## Citation Rules

- Every factual claim drawn from a raw source should reference it: `(source: filename)`
- If two sources disagree, note the contradiction explicitly
- If a claim has no source, mark it `%%needs source%%`
- Decisions and specs do not require citations but must link to relevant wiki pages

---

## Lint

When the user asks you to lint or audit the vault:

- Check for contradictions between pages
- Find orphan pages (no inbound links)
- Identify concepts mentioned in pages that lack their own page
- Flag claims marked `%%needs source%%`
- Check that all pages follow the correct format for their type
- Verify `wiki/index.md` is complete and up to date
- Report findings as a numbered list with suggested fixes

---

## Rules

- **Never modify** anything in the `raw/` folder
- **Always update** `wiki/index.md`, `wiki/hot.md`, and `wiki/log.md` after any change
- Use `[[wikilinks]]` for all internal vault references — never relative file paths
- Keep file names lowercase with hyphens (e.g. `commission-model.md`)
- Decision files are prefixed with date: `YYYY-MM-DD-title.md`
- Write in clear, plain language — this vault is a working tool, not a showcase
- When uncertain about how to categorize something, ask the user
- The platform name is currently **YachtBay** — this may change; update vault-wide if it does
