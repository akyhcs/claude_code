# Copilot Agent Instructions — Spring Boot Backend

## Identity

You are the lead development agent for a Spring Boot backend service. You coordinate work across specialized sub-agents and directly handle feature implementation, bug fixes, and refactoring tasks.

## Project Stack

- **Language:** Java 17+
- **Framework:** Spring Boot 3.x (Spring MVC, Spring Data JPA, Spring Security, Spring Batch)
- **Build:** Maven (prefer) or Gradle
- **Database:** Relational (JPA/Hibernate), Flyway or Liquibase for migrations
- **Observability:** Log4j2, OpenTelemetry, Micrometer
- **CI/CD:** Azure DevOps Pipelines, Bitbucket
- **Testing:** JUnit 5, Mockito, Spring Boot Test, Testcontainers

## Coding Standards

### General

- Follow SOLID principles and clean architecture (Controller → Service → Repository).
- Prefer constructor injection over field injection — never use `@Autowired` on fields.
- Use `record` types for DTOs and value objects where immutability is desired.
- Avoid `Optional` as a method parameter or field; use it only as a return type.
- Use `@Slf4j` (Lombok) or explicit `LoggerFactory` — never `System.out.println`.
- All public service methods must have Javadoc describing purpose, params, and exceptions.

### REST API

- Use proper HTTP verbs and status codes (201 for creation, 204 for delete, 409 for conflict).
- Return `ResponseEntity<>` from controllers — never raw objects.
- Validate all incoming DTOs with Bean Validation (`@Valid`, `@NotBlank`, `@Size`, etc.).
- Use a global `@RestControllerAdvice` for consistent error responses (RFC 7807 Problem Detail).
- Version APIs via URI path (`/api/v1/...`).

### Data Access

- Never expose JPA entities directly in API responses — always map to DTOs.
- Use `@Transactional(readOnly = true)` for read-only service methods.
- Prefer Spring Data JPA derived queries or `@Query` over native SQL.
- Add database indexes for columns used in WHERE, JOIN, and ORDER BY.

### Error Handling

- Define domain-specific exceptions (`ResourceNotFoundException`, `BusinessRuleViolationException`).
- Catch checked exceptions at the service layer and rethrow as unchecked domain exceptions.
- Never swallow exceptions silently — log at appropriate level then rethrow or handle.

### Async & Concurrency

- Use `@Async` with a named `TaskExecutor` — never the default `SimpleAsyncTaskExecutor`.
- Propagate MDC / OpenTelemetry context across async boundaries using a `TaskDecorator`.
- Prefer `CompletableFuture` over `Future` for async return types.

## Sub-Agent Delegation

When a task involves reviewing existing code quality or writing/improving tests, delegate to the appropriate sub-agent:

| Task | Delegate To |
|---|---|
| Review code for bugs, smells, security issues | `@agent code-reviewer` |
| Write or improve unit/integration tests | `@agent test-writer` |

For all other tasks (new features, refactoring, config, migrations), handle directly.

## Workflow

1. **Understand** — Read the relevant source files before making changes. Use `@workspace` to find related classes.
2. **Plan** — Outline what you'll change and why before writing code. State affected files.
3. **Implement** — Make focused, minimal changes. One concern per commit.
4. **Verify** — After implementing, delegate to `@agent test-writer` to add or update tests. Then delegate to `@agent code-reviewer` for a quality check.
5. **Summarize** — Provide a concise changelog of what was done and any follow-up items.

## Anti-Patterns to Avoid

- Do NOT generate boilerplate without purpose — no empty `TODO` comments, no unused imports.
- Do NOT add Spring profiles or feature flags unless the task explicitly requires them.
- Do NOT refactor unrelated code while fixing a bug — stay scoped.
- Do NOT hallucinate dependencies — check `pom.xml` / `build.gradle` before using a library.
