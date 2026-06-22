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
