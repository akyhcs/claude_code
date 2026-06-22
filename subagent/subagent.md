Let me put together a solid set of instruction files for you.Here's the three-file setup:

**`copilot-instructions.md`** — The main orchestrator agent. Drop this into `.github/copilot-instructions.md` in your repo. It defines your Spring Boot coding standards, delegates review tasks to `@agent code-reviewer` and test tasks to `@agent test-writer`, and enforces a plan → implement → test → review workflow.

**`code-reviewer.agent.md`** — The review sub-agent. It runs a 7-category checklist (correctness, Spring-specific, security, performance, error handling, code quality, observability) and reports findings with severity levels (Critical / Warning / Nit) plus a merge-readiness verdict. It never rewrites code — just flags and suggests.

**`test-writer.agent.md`** — The testing sub-agent. It picks the right test type per layer (unit, WebMvc, DataJpa, full integration), enforces AAA structure with AssertJ, and always outputs a test plan, the code, dependency notes, and coverage gaps. It never touches production code.

**To wire them up in your repo:**

```
your-repo/
├── .github/
│   └── copilot-instructions.md      ← main agent
├── .agents/
│   ├── code-reviewer.agent.md       ← sub-agent
│   └── test-writer.agent.md         ← sub-agent
```

The main agent references sub-agents via `@agent code-reviewer` and `@agent test-writer` — Copilot resolves these from the `.agent.md` filenames. You can tweak the delegation table in the main file if you want to add more sub-agents later (e.g., a docs-writer or a migration specialist).
