# Setup Guide — Centralized Copilot Agent Architecture

## Architecture Overview

```
┌──────────────────────────────────────────────────────────┐
│  HOME DIRECTORY (user-level — highest priority)          │
│  Universal standards, reusable across ALL repos          │
│                                                          │
│  ~/copilot-prompts/                                      │
│  ├── copilot-instructions.md   ← coding standards        │
│  ├── code-reviewer.agent.md    ← review sub-agent        │
│  └── test-writer.agent.md      ← testing sub-agent       │
└──────────────────────────────────────────────────────────┘
                     ↓ merges with ↓
┌──────────────────────────────────────────────────────────┐
│  EACH REPO (project-level — merges below user-level)     │
│  Only project-specific context                           │
│                                                          │
│  your-repo/.github/                                      │
│  └── copilot-instructions.md   ← DB, entities, rules     │
└──────────────────────────────────────────────────────────┘
```

## Step 1 — Create the Home Prompt Repo

Store your universal prompts in a dedicated folder (or a Git repo you clone to every machine):

```bash
mkdir -p ~/copilot-prompts
# Copy the three home-level files into this folder:
# - copilot-instructions.md
# - code-reviewer.agent.md
# - test-writer.agent.md
```

Optionally, make it a Git repo for versioning:

```bash
cd ~/copilot-prompts
git init
git add -A
git commit -m "Initial copilot prompt library"
```

## Step 2 — Wire into VS Code

Open VS Code `settings.json` (Ctrl+Shift+P → "Preferences: Open User Settings (JSON)") and add:

```jsonc
{
  // Points Copilot to your universal instruction file
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": "~/copilot-prompts/copilot-instructions.md"
    }
  ]
}
```

> **Note on .agent.md files:** As of mid-2025, VS Code resolves `.agent.md`
> files from the workspace root, not from user settings. To make the sub-agents
> available in every project, use ONE of these strategies:
>
> **Option A — Symlinks (simplest)**
> ```bash
> # Run once per repo
> cd your-repo
> mkdir -p .github/agents
> ln -s ~/copilot-prompts/code-reviewer.agent.md .github/agents/code-reviewer.agent.md
> ln -s ~/copilot-prompts/test-writer.agent.md   .github/agents/test-writer.agent.md
> ```
> Add `.github/agents/*.agent.md` to `.gitignore` if you don't want them committed.
>
> **Option B — Git submodule**
> ```bash
> # If copilot-prompts is a remote repo
> cd your-repo
> git submodule add git@github.com:yourorg/copilot-prompts.git .github/agents
> ```
>
> **Option C — Multi-root workspace**
> Add `~/copilot-prompts` as a second folder in your VS Code workspace.
> Copilot will scan both roots for `.agent.md` files.

## Step 3 — Wire into IntelliJ IDEA

1. Open **Settings → Tools → GitHub Copilot → Chat**
2. Under **Code Generation Instructions**, add the path:
   `~/copilot-prompts/copilot-instructions.md`
3. For `.agent.md` sub-agents, use the symlink strategy (Option A above)
   since IntelliJ also resolves agents from the project root.

## Step 4 — Add Project-Level Config

In each repo, create a thin `.github/copilot-instructions.md` with ONLY:

- Stack specifics (Java version, DB, broker, auth)
- Key entities and domain rules
- API paths and naming conventions
- CI/CD notes

**Do NOT duplicate** coding standards, review rules, or test patterns here —
those come from the home-level files automatically.

## How Precedence Works

```
Priority (highest → lowest):
  1. Your chat prompt (always wins)
  2. User-level instructions (~/copilot-prompts/)
  3. Repo-level instructions (.github/)
  4. Org-level instructions (Enterprise/Business settings)
```

All tiers MERGE additively. Precedence only matters when two instructions
directly contradict each other — the higher tier wins. If they don't conflict
(and they shouldn't if you follow this pattern), both load together.

## Keeping It Clean — The Split Rule

| Goes in HOME (universal)              | Goes in REPO (project-specific)          |
|---------------------------------------|------------------------------------------|
| Java/Spring coding standards          | Java version, Spring Boot version         |
| REST API conventions                  | API base paths, endpoint inventory        |
| Error handling patterns               | Domain exception classes                  |
| Test structure (AAA, naming, mocking) | Test DB config (Testcontainers image)     |
| Review checklist categories           | Entity relationships, business rules      |
| Async/concurrency rules              | Executor bean names, queue names          |
| Observability standards               | Specific metric names, dashboard links    |
| Sub-agent personas & checklists       | Delegation table (if project adds agents) |
