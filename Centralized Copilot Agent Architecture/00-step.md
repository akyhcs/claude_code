That's the right call — it aligns perfectly with how Copilot's precedence works. User-level (home) loads on every project as your baseline, repo-level (`.github/`) adds only what's unique. They merge additively.Here's how that layering works with Copilot's precedence system.Now let me restructure the actual files to match this architecture. The home-level files carry your universal standards, and the project-level file is just a thin config layer.Now the thin project-level files — these go in each repo's `.github/`:Here's the full package — 5 files total:

**SETUP-GUIDE.md** — Start here. Covers the wiring for both VS Code and IntelliJ, including three strategies to make home-level `.agent.md` files visible in each repo (symlinks, git submodule, or multi-root workspace).

**Home-level files** (go in `~/copilot-prompts/`, loaded on every project):

- `home-copilot-instructions.md` → Rename to `copilot-instructions.md` when placing. Universal Spring Boot standards — architecture, REST conventions, error handling, async rules. No project-specific details.
- `code-reviewer.agent.md` → 7-category checklist with severity levels. Finds issues, never rewrites.
- `test-writer.agent.md` → Picks the right test type per layer, enforces AAA + AssertJ, outputs a test plan with coverage gaps.

**Project-level file** (goes in each repo's `.github/`):

- `project-copilot-instructions.md` → Rename to `copilot-instructions.md` when placing. This is a sample for an Order Management Service — just stack versions, entities, domain rules, and API paths. Swap in your real project's details.

The key principle: if it applies to every Spring Boot project you work on, it goes in the home directory. If it only makes sense for *this* repo, it goes in `.github/`. They merge together at runtime with no conflicts since they cover different concerns.
