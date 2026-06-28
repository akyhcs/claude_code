Clean folder structure for an agent setup, focused on the three things you asked about:

```
my-agent/
│
├── AGENTS.md                          # Top-level: who this agent is, what it does
│
├── agents/                            # Specialized agents (one .md per agent)
│   ├── code-reviewer.agent.md
│   ├── terraform-helper.agent.md
│   └── docs-writer.agent.md
│
├── skills/                            # Focused expertise (one folder per skill)
│   │
│   ├── spring-boot-errors/
│   │   ├── SKILL.md                   # Required — frontmatter + body
│   │   ├── references/                # Loaded only when SKILL.md says to
│   │   │   ├── validation.md
│   │   │   └── advanced-patterns.md
│   │   ├── scripts/                   # Executable helpers
│   │   │   └── check_handler.py
│   │   └── assets/                    # Templates the skill produces
│   │       └── handler-template.java
│   │
│   ├── terraform-azure-module/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── networking.md
│   │
│   └── python-testing/
│       └── SKILL.md
│
├── docs/                              # Knowledge sources the agent reads
│   ├── architecture.md                # System design, decisions
│   ├── conventions.md                 # Coding standards, patterns
│   ├── glossary.md                    # Domain terms
│   ├── api-specs/                     # External API contracts
│   │   └── docusign-webhook.yaml
│   └── wiki/                          # Synced from Confluence/Notion
│       ├── runbooks.md
│       └── postmortems.md
│
├── prompts/                           # Reusable prompt templates
│   ├── new-feature.md
│   └── pr-review.md
│
└── mcp-config.json                    # Tools (MCP servers)
```

**What each thing is, briefly:**

**`AGENTS.md`** — single file at the root. Defines the agent's identity, scope, and what it refuses. Read first, applies to everything.

**`agents/`** — specialized variants. One `.agent.md` per role. Each file declares its purpose, which skills it uses, which tools it can call. A `code-reviewer.agent.md` might pull in the `spring-boot-errors` skill plus the GitHub MCP tool only.

**`skills/`** — one folder per skill. Each must contain a `SKILL.md` with frontmatter (`name`, `description`) plus the body. The folder can also have:
- `references/` — deeper documentation, loaded on demand (progressive disclosure)
- `scripts/` — executable helpers for repetitive deterministic work
- `assets/` — templates, fixtures, anything the skill outputs

**`docs/`** — flat or nested, doesn't matter. Anything that's *content* the agent reads as context: architecture notes, conventions, API specs, wiki syncs. The agent loads these either by being told to read them, or by an MCP server exposing a `search_docs` tool over them.

**`prompts/`** — reusable prompt templates the user invokes by name (e.g. `/pr-review`).

**`mcp-config.json`** — declares which MCP servers (tools) this agent has access to. Lives at the root.

**Naming conventions worth following:**

- Skill folders in `kebab-case`, matching the `name:` field in their SKILL.md
- Agent files end in `.agent.md`
- Skill files always named `SKILL.md` (uppercase)
- Docs in plain `.md` (no special suffix needed)

That's the whole structure. Three things, clear separation: **agents** decide and orchestrate, **skills** encode know-how, **docs** carry facts.
