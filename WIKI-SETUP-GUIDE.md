# Implementation Guide: Karpathy Skills + Obsidian Wiki System

## Part 1 — What This Repo Actually Is

The `andrej-karpathy-skills` repo is a **two-layer behaviour system for Claude Code**:

| Layer | File | Scope | Purpose |
|---|---|---|---|
| Plugin | `skills/karpathy-guidelines/SKILL.md` | Global (all projects) | Reusable skill, installed once |
| Per-project | `CLAUDE.md` | Single project | Merged project-specific rules |

The `SKILL.md` and `CLAUDE.md` are nearly identical in content — the difference is *how they are loaded*. A SKILL is injected automatically via the plugin system every session. A `CLAUDE.md` is checked into a repo and only applies there.

---

## Part 2 — How This Aligns with Karpathy's LLM Wiki Approach

Karpathy's core insight (from the tweet this repo references) is that LLMs fail in predictable, repeatable ways:

- They assume silently instead of surfacing confusion
- They over-engineer instead of doing the minimum
- They "drift" into adjacent code they were never asked to touch
- They accept vague goals instead of demanding verifiable criteria

His prescription is essentially **treat the LLM like a new hire who needs a written wiki** — not code comments, but a standing document that says *how to behave in this codebase, on this team, with these constraints*.

This repo operationalises that by turning his tweet into a structured `CLAUDE.md` / `SKILL.md`. The alignment is direct:

| Karpathy Observation | Repo Principle |
|---|---|
| "Make wrong assumptions, run with them" | Think Before Coding |
| "Overcomplicate, bloat abstractions" | Simplicity First |
| "Change/remove code they don't understand" | Surgical Changes |
| "LLMs excel at looping to meet specific goals" | Goal-Driven Execution |

The deeper Karpathy wiki idea is that **context fed to the LLM at session start is your most powerful lever** — more impactful than any individual prompt. This repo makes that lever permanent and reusable.

---

## Part 3 — Step-by-Step Setup for Your Sidegig Project

### Step 1: Install the Plugin Globally (do once)

In any Claude Code session:

```
/plugin marketplace add forrestchang/andrej-karpathy-skills
/plugin install andrej-karpathy-skills@karpathy-skills
```

This installs the `karpathy-guidelines` skill globally. It will inject into every Claude Code session automatically — you never touch it again.

**Verify it worked:** Open a new session and ask Claude "what skills do you have?" — it should mention karpathy-guidelines.

### Step 2: Add a Project-Specific CLAUDE.md to Your Sidegig

In your sidegig project root:

```bash
# Option A — download the base (if you want a copy independent of the plugin)
curl -o CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md

# Option B — create a minimal CLAUDE.md that adds only project-specific rules
# (the plugin already handles the 4 principles globally)
touch CLAUDE.md
```

For Option B (recommended when the plugin is installed), your `CLAUDE.md` only needs the project-specific section:

```markdown
# Project: [Sidegig Name]

## Stack
- [e.g., Next.js 14, Supabase, Tailwind]

## Project-Specific Rules
- All API routes live in `src/app/api/` — follow existing patterns before adding new ones
- Auth is handled by [X] — never roll custom session logic
- Database schema is in `supabase/migrations/` — always add a migration for schema changes

## Known Gotchas
- [e.g., The `useUser` hook returns null on first render — always guard with a loading state]
```

The plugin injects the 4 principles globally; your `CLAUDE.md` adds the *why this codebase is different*.

### Step 3: Verify the Layering Works

Open Claude Code in your sidegig project and test:

- Ask something ambiguous → Claude should ask a clarifying question (Think Before Coding)
- Ask for a "quick feature" → Claude should do the minimum, not over-engineer (Simplicity First)
- Ask to fix a bug → Claude should only touch the bug, not reformat surrounding code (Surgical Changes)

---

## Part 4 — Central Obsidian Wiki + Project-Specific Wikis

The goal: a **two-tier knowledge system** where common patterns live in one place and project context lives per-project, and both feed directly into Claude Code sessions.

### Architecture

```
~/obsidian-vault/
├── _shared/                    ← Central wiki (common patterns, solutions)
│   ├── common-bugs.md
│   ├── reusable-patterns.md
│   ├── tech-stack-notes.md
│   └── claude-context.md       ← Exports to CLAUDE.md snippets
├── projects/
│   ├── sidegig/
│   │   ├── architecture.md
│   │   ├── known-issues.md
│   │   └── claude-context.md   ← Project-specific CLAUDE.md content
│   └── other-project/
│       └── ...
└── MEMORY.md                   ← Index (mirrors Claude's memory index)
```

### Step 1: Set Up Obsidian Vault Structure

1. Create a new Obsidian vault (or use existing one)
2. Create the folder structure above
3. Install these Obsidian plugins (Community Plugins):
   - **Dataview** — query across notes (find all "known bugs" tagged `#gotcha`)
   - **Templater** — standardise note structure
   - **Git** (optional) — sync vault to a private repo

### Step 2: Create Note Templates

**Template: `_templates/project-issue.md`**
```markdown
---
tags: [#gotcha, #{{project}}]
date: {{date}}
---

## Issue: {{title}}

**Symptom:** 

**Root cause:** 

**Fix:** 

**Claude context snippet:**
\```
- [Gotcha] {{title}}: <one-line rule for CLAUDE.md>
\```
```

**Template: `_templates/reusable-pattern.md`**
```markdown
---
tags: [#pattern, #{{stack}}]
---

## Pattern: {{title}}

**When to use:** 

**Code:**

**Claude context snippet:**
\```
- Follow the {{title}} pattern in `path/to/example.ts`
\```
```

The **Claude context snippet** field is the key habit: every note you write produces a one-liner you can paste into a `CLAUDE.md`.

### Step 3: Build the Central `_shared/claude-context.md`

This is a living file you paste from your notes into shared `CLAUDE.md` snippets:

```markdown
# Shared Claude Context

## Common Gotchas Across Projects
- Never use `date-fns` and `dayjs` in the same project — pick one per codebase
- Supabase RLS policies don't apply to service-role clients — always check which client is in use
- React Query's `staleTime: 0` default causes double-fetching — always set it explicitly

## Reusable Patterns
- Auth: use the pattern in `_shared/reusable-patterns/supabase-auth.md`
- Error boundaries: see `_shared/reusable-patterns/error-boundary.md`
- API response shape: `{ data, error, meta }` — never diverge from this

## Stack Preferences
- ORM: Prisma over raw SQL unless performance-critical
- State: Zustand for global, React Query for server, useState for local — no exceptions
```

### Step 4: Link Obsidian to Your Project CLAUDE.md Files

**Manual workflow (simplest):**
1. When you hit a new issue in sidegig, write a note in `projects/sidegig/known-issues.md`
2. Extract the Claude context snippet from the note
3. Paste it into `sidegig/CLAUDE.md` under "Known Gotchas"

**Semi-automated workflow (Obsidian + a script):**

Create a script `scripts/sync-claude-context.sh` in each project:

```bash
#!/bin/bash
# Sync Claude context from Obsidian into CLAUDE.md
# Run this manually before a Claude Code session when you've updated the wiki

VAULT="$HOME/obsidian-vault"
PROJECT_CONTEXT="$VAULT/projects/sidegig/claude-context.md"
SHARED_CONTEXT="$VAULT/_shared/claude-context.md"

# Strip frontmatter and append to project CLAUDE.md
# (Only updates the dynamic section — keep your static rules separate)
echo "" >> CLAUDE.md
echo "## Context from Wiki (auto-synced)" >> CLAUDE.md
grep -v "^---" "$SHARED_CONTEXT" >> CLAUDE.md
grep -v "^---" "$PROJECT_CONTEXT" >> CLAUDE.md

echo "CLAUDE.md updated from wiki"
```

**Fully automated (Obsidian Git + watch):**

If you keep the vault in git, you can wire a `post-commit` hook in the vault repo that runs the sync script for all projects. More complex, but means wiki edits instantly propagate to `CLAUDE.md` files.

### Step 5: Establish the Habit Loop

The system only works if you feed it. The loop:

```
1. Hit a bug or pattern in a project
      ↓
2. Write a note in Obsidian (use the template — takes 3 min)
      ↓
3. Extract the "Claude context snippet" from the note
      ↓
4. Paste into project CLAUDE.md + _shared/claude-context.md if reusable
      ↓
5. Next Claude session has the context without you re-explaining
```

The forcing function: **never explain the same thing to Claude twice**. If you catch yourself re-explaining a gotcha or preference, that's a signal to write the note first.

---

## Part 5 — Full Setup Checklist

```
Global setup (once):
[ ] Install karpathy-guidelines plugin in Claude Code
[ ] Create Obsidian vault with _shared/ and projects/ structure
[ ] Install Dataview + Templater in Obsidian
[ ] Create issue and pattern templates

Per new project:
[ ] Create CLAUDE.md with project-specific rules (not the 4 principles — plugin handles those)
[ ] Create projects/<name>/ folder in Obsidian vault
[ ] Create projects/<name>/claude-context.md
[ ] Link sync script if desired

Ongoing habit:
[ ] New issue hit → write Obsidian note → extract snippet → update CLAUDE.md
[ ] Monthly: review CLAUDE.md for stale entries, prune dead rules
[ ] Quarterly: review _shared/claude-context.md, promote project-specific patterns that appear in 2+ projects
```

---

## Summary of the System

```
Obsidian vault          →  source of truth for what you know
claude-context.md       →  distilled rules extracted from wiki notes
CLAUDE.md (project)     →  feeds Claude context at session start
karpathy-guidelines     →  global behavioural baseline (always active)
Claude Code session     →  all layers combined: baseline + project rules + wiki context
```

The net effect: Claude Code behaves like a developer who has already read your runbook, knows your known bugs, follows your stack preferences, and never needs the same correction twice.
