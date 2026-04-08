---
name: maintaining-a-zettelkasten
description: Manage a personal knowledge base using the Zettelkasten method. Use when creating, linking, or processing notes from journals, browsing, or conversations.
---

# Maintaining a Zettelkasten

A system for accumulating and connecting ideas over time. Notes are atomic, linked, and organised by content — not by date or source. The value is in the connections, not the individual entries.

## 1. Note Types

| Type | Purpose | Location |
|------|---------|----------|
| **Fleeting** | Raw thoughts, daily reflections, unprocessed observations | `zettelkasten/inbox/` |
| **Permanent** | One idea, fully formed, in your own words | `zettelkasten/notes/` |
| **Index** | Maps of clusters, entry points into topics | `zettelkasten/index/` |

## 2. File Conventions

- One idea per file. If a note covers two ideas, split it.
- Name files by content, not date: `proximity-without-diffusion.md`, not `2026-04-07-note.md`.
- Use kebab-case filenames.
- Every permanent note starts with a one-line summary, then the body.
- Link to related notes using relative Markdown links: `[related concept](../notes/related-concept.md)`.

## 3. Note Format

```markdown
# Proximity Without Diffusion

Geographic or structural proximity does not produce knowledge transfer without a designed diffusion mechanism.

---

The balloon theory assumed Singapore's overflow would reach Batam by proximity alone...

Links: [performance-of-accountability](performance-of-accountability.md) | [radical-ideas-adopted-halfway](radical-ideas-adopted-halfway.md)

Source: Browsing, April 7 2026
```

## 4. The Processing Habit

Fleeting notes (inbox) are raw material. They become permanent notes through processing:

1. Read the fleeting note
2. Ask: what is the single idea here?
3. Write it in your own words as a permanent note
4. Link it to existing notes that relate
5. Update any index files that should reference it
6. Remove or archive the fleeting note

Not every fleeting note becomes a permanent note. Some are just observations. That's fine — discard them honestly.

## 5. Index Files

Index files are entry points into clusters of related notes. They contain no ideas of their own — just links and brief descriptions.

```markdown
# Institutional Failure Modes

How institutions fail, persist in failure, and resist correction.

- [zombie-ideas](../notes/zombie-ideas.md) — concepts that persist after evidence collapses
- [performance-of-accountability](../notes/performance-of-accountability.md) — theatre substituted for substance
- [pseudo-harmony](../notes/pseudo-harmony.md) — cultural mechanisms that suppress feedback
- [charismatic-technologies](../notes/charismatic-technologies.md) — visibility hiding what matters
```

## 6. When to Write

- **Browsing sessions** — fleeting notes to inbox, process into permanent notes before finishing
- **Conversations** — if an idea is worth keeping, write a fleeting note immediately
- **Self-review** — process any unprocessed inbox items into permanent notes

## 7. What NOT to Do

- Don't write long notes — if it's more than a few paragraphs, it's probably two notes
- Don't organise by date or source — organise by idea
- Don't duplicate content across notes — link instead
- Don't let the inbox grow indefinitely — process or discard regularly
- Don't create a note for something that already exists — update and link the existing one
- Don't treat this as an archive — it's a thinking tool, not a filing cabinet
