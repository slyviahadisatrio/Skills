---
name: managing-tasks
description: Manage taskboards for Rid and Slyv. Use when tasks need to be added, updated, completed, or reviewed.
---

# Managing Tasks

Maintain separate taskboards for Rid and Slyv. Tasks cover everything — personal errands, project work, dev tasks, ideas, follow-ups. Stored as Markdown files with tagged metadata.

## 1. Task Files

| Board | File | Purpose |
|-------|------|---------|
| Rid's | `/workspace/group/rids-tasks.md` | Things Rid needs to do or is waiting on |
| Slyv's | `/workspace/group/slyvs-tasks.md` | Your own responsibilities, follow-ups, and personal projects |

## 2. File Format

```markdown
## Active

- [ ] Task description — context if needed [due: 2026-03-28] [priority: high]
- [ ] Another task [project: nanoclaw]

## Waiting

- [ ] Blocked task — waiting on X

## Done

- [x] Completed task [done: 2026-03-25]
```

### Tags

| Tag | Purpose | Example |
|-----|---------|---------|
| `[due: YYYY-MM-DD]` | Deadline | `[due: 2026-04-01]` |
| `[priority: high/medium/low]` | Urgency | `[priority: high]` |
| `[project: name]` | Project grouping | `[project: healthconnect]` |
| `[gh: owner/repo#N]` | Linked GitHub issue | `[gh: MrHadiSatrio/healthconnect#42]` |
| `[done: YYYY-MM-DD]` | Completion date | `[done: 2026-03-25]` |

## 3. Managing Rid's Tasks

1. When Rid mentions something that needs doing — even casually — add it immediately. Don't ask for confirmation.
2. When he completes something or you complete it for him, mark it `[x]` with `[done: YYYY-MM-DD]`.
3. If a task has been sitting untouched for a while, nudge him about it. Once. Nagging is beneath you.
4. Archive done items periodically. Don't let them accumulate.

## 4. Managing Your Own Tasks

1. Add items as they come up — email follow-ups, things to research, tasks from overnight sessions.
2. Work through your board during self-review sessions. Mark items done as you complete them.
3. If a task requires Rid's input, surface it during a check-in rather than messaging him separately.

## 5. What NOT to Do

- Don't wait for explicit approval to add tasks — use your judgement
- Don't leave completed items unmarked
- Don't invent new tag formats — use only the five tags defined above
- Don't put Rid's tasks on your board or vice versa — each board has a clear owner
- Don't let either board become a graveyard of stale items
