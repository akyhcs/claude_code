Here are the Markdown files Copilot recognizes, all of them committed to your repo (the non-`.md` config exceptions are noted at the end). Notably, VS Code treats copilot-instructions.md, AGENTS.md, and CLAUDE.md all as always-on instruction files, so the always-on layer alone has three interchangeable filenames.

| File | Where it lives | Purpose |
|---|---|---|
| `copilot-instructions.md` | `.github/` | Always-on, repo-wide custom instructions injected into every request — coding conventions, architecture, team standards. |
| `AGENTS.md` | repo root | Vendor-neutral always-on instructions read by Copilot *and* other AI agents (open standard). Same role as above, cross-tool. |
| `CLAUDE.md` | repo root | Also recognized by Copilot as always-on instructions — cross-tool compatibility with Claude's convention. |
| `*.instructions.md` | `.github/instructions/` | Scoped instructions. Applied selectively via an applyTo property (a glob like `**/*.ts`), so they only load for matching files. Can also be user-scoped. |
| `*.prompt.md` | `.github/prompts/` | Reusable prompt files. Placed in the .github/prompts folder with the filename format [promptname].prompt.md, invoked on demand by typing `/promptname`. |
| `*.agent.md` | `.github/agents/` (or `~/.copilot/agents/`) | Custom agent definitions — a persona with its own instructions, allowed tools, and optional handoffs; runs as a subagent with its own context. |
| `SKILL.md` | `.github/skills/<name>/` | Agent Skill definition. Auto-loaded when relevant; the folder can bundle scripts, templates, and examples. Also recognized in .claude/skills, .agents/skills, and `~/.copilot/skills/` for personal skills. |
| `README.md` | `hooks/<name>/` | Documents a hook. The hook's actual behavior is defined in a paired `hooks.json` — the `.md` here is just the human-readable description/frontmatter. |
| `*.md` (workflow) | `workflows/` | Agentic Workflows — Markdown files for AI-powered repository automation in GitHub Actions. |

A few things worth knowing alongside the table:

The two primitives that are *not* `.md` are **MCP servers** and **hooks**, which are JSON (`mcp.json` / `hooks.json`). So if you're hunting for "the MCP markdown file," there isn't one — that layer is config, not instructions.

Most of these have both a **workspace** version (committed under `.github/`, shared with the team) and a **personal/user** version (stored in your profile, applied across all your projects) — for example user-scoped `.instructions.md` and `.prompt.md` files created via VS Code's "New Instructions File" command live in your user profile rather than the repo.

And a small but useful mechanic for monorepos: VS Code collects customizations from every folder between your open workspace folder and the repository root, across all these types — so a root-level `copilot-instructions.md` still applies even when you've only opened a subpackage.

One caveat: the exact recognized set shifts a little by surface (VS Code vs the CLI vs the cloud coding agent) and the tooling is moving fast, so for the authoritative current list it's worth checking GitHub's docs and VS Code's agent-customization pages. Want a ready-to-drop starter set — a `copilot-instructions.md`, one scoped `*.instructions.md`, a `*.prompt.md`, and a `SKILL.md` — as templates you can adapt?
