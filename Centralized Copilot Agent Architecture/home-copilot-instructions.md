# Universal Copilot Instructions — Java / Spring Boot Standards

> **Scope:** User-level. Loaded on every project via VS Code / IntelliJ settings.
> This file contains stack-agnostic and Spring Boot standards that apply everywhere.
> Project-specific details (DB, endpoints, schema) belong in the repo's `.github/copilot-instructions.md`.

## Language & Framework Defaults

- Java 17+ with Spring Boot 3.x
- Prefer constructor injection — never `@Autowired` on fields
- Use `record` types for DTOs and value objects
- Use `@Slf4j` (Lombok) or explicit `LoggerFactory` — never `System.out.println`
- All public service methods must have Javadoc

## Architecture

- Follow Controller → Service → Repository layering
- Keep controllers thin — no business logic, only request mapping + validation + delegation
- Services are transactional boundaries — catch exceptions here, not in controllers
- Never expose JPA entities in API responses — always map to DTOs

## REST API Conventions

- Proper HTTP verbs and status codes (201 create, 204 delete, 409 conflict)
- Return `ResponseEntity<>` from controllers
- Validate incoming DTOs with `@Valid` + Bean Validation annotations
- Use `@RestControllerAdvice` for RFC 7807 Problem Detail error responses
- Version APIs via URI path: `/api/v1/...`

## Data Access

- `@Transactional(readOnly = true)` for read-only service methods
- Prefer Spring Data JPA derived queries or `@Query` — avoid native SQL
- Add indexes for columns used in WHERE, JOIN, ORDER BY

## Error Handling

- Define domain exceptions: `ResourceNotFoundException`, `BusinessRuleViolationException`, etc.
- Catch checked exceptions at service layer → rethrow as unchecked domain exceptions
- Never swallow exceptions — log at appropriate level then rethrow or handle

## Async & Concurrency

- Use `@Async` with a named `TaskExecutor` — never the default `SimpleAsyncTaskExecutor`
- Propagate MDC / OpenTelemetry context via `TaskDecorator`
- Prefer `CompletableFuture` over `Future`

## Observability

- Structured logging with MDC correlation IDs
- OpenTelemetry spans on critical paths
- Micrometer metrics for business KPIs

## Anti-Patterns

- No boilerplate without purpose — no empty TODOs, no unused imports
- No unrelated refactoring during bug fixes — stay scoped
- Do not hallucinate dependencies — check `pom.xml` / `build.gradle` first

# Project: Order Management Service

> **Scope:** Repo-level. Only project-specific context lives here.
> Universal coding standards, review rules, and test patterns are inherited
> from user-level instructions (loaded automatically by the IDE).

## Stack

- **Java:** 21 (virtual threads enabled)
- **Spring Boot:** 3.3.x
- **Database:** PostgreSQL 15 via Spring Data JPA
- **Migrations:** Flyway — naming convention: `V{yyyyMMdd_HHmm}__{description}.sql`
- **Message broker:** Azure Service Bus (via `spring-cloud-azure-starter-servicebus`)
- **Auth:** Spring Security + OAuth2 Resource Server (Azure AD tokens)
- **Build:** Maven multi-module (`:api`, `:domain`, `:infra`)

## Key Entities

- `Order` → `OrderItem` (1:N)
- `Customer` (linked via `customerId`, lives in a separate service)
- `Inventory` (checked via REST call to Inventory Service)

## API Base Path

`/api/v1/orders`

## Environment-Specific Notes

- Local dev uses Testcontainers for PostgreSQL (no local DB install needed)
- `application-local.yml` has Service Bus connection string for local emulator
- CI pipeline runs in Azure DevOps — see `azure-pipelines.yml`

## Domain Rules

- An order cannot be placed if inventory is insufficient (call Inventory Service)
- Orders in `SHIPPED` status cannot be cancelled
- Refunds are only allowed within 30 days of delivery
- All monetary amounts use `BigDecimal` — never `double` or `float`

## Sub-Agent Delegation

The code-reviewer and test-writer agents are loaded from user-level settings.
Invoke them as usual:

| Task | Delegate |
|---|---|
| Review a PR or changeset | `@agent code-reviewer` |
| Write or improve tests | `@agent test-writer` |

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
