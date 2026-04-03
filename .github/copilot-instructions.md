# Copilot Instructions

This is a GitHub [`.github` special repository](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) for the `shawnewallace` GitHub account. Files placed here serve as **defaults** for all repositories under the account that don't define their own versions.

## Repository Purpose

This repository holds default community health files and shared GitHub configuration:

- **Community health files**: `CODE_OF_CONDUCT.md`, `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md`
- **Issue/PR templates**: `.github/ISSUE_TEMPLATE/`, `.github/PULL_REQUEST_TEMPLATE.md`
- **GitHub Actions workflows** (reusable or defaults)
- **Copilot instructions**: `.github/copilot-instructions.md` (this file)

Files at the root are visible to GitHub as default community health files. Keep them general enough to apply across all repos in the account.

---

## Primary Stack

C# / .NET is the primary language and platform. Other languages appear occasionally.

---

## Workflow for Implementing Features

1. Identify and list any relevant instruction files before starting.
2. Follow **TDD**: write or update tests first, then implement.
3. Run `dotnet test` (or `dotnet build`) to verify all tests pass before committing. Run it — don't ask.
4. Fix all compiler warnings and errors before moving to the next step.

---

## Architecture: Clean Architecture + DDD

### Solution Structure

Four projects per layer:

```
src/
  [Project].Domain/          # Entities, value objects, domain events — no dependencies
  [Project].Application/     # Use cases, commands, queries, interfaces — depends on Domain only
  [Project].Infrastructure/  # DB, external services — depends on Application + Domain
  [Project].Api/             # Minimal API endpoints — depends on Infrastructure only
tests/
  [Project].UnitTests/       # Domain + Application layers
  [Project].IntegrationTests/ # Infrastructure + Api layers; includes ArchitectureTests.cs
```

- Use **feature-oriented folders** (e.g., `Order/`, `Customer/`), not technical folders (e.g., `Entities/`, `Services/`).
- Each project has a marker file (e.g., `DomainReference.cs`) for architecture test discovery.
- Enforce layer dependencies with **NetArchTest** in `ArchitectureTests.cs`.
- Do **not** use a mediator library (no MediatR); call service methods directly from the Api layer.

### Domain Modeling (DDD)

- **Aggregates**: Always created via static factory method (e.g., `Order.Create(...)`). Never via public constructor. Private/protected constructors only. Enforce all invariants within the boundary.
- **Entities**: Owned by aggregates. Created via the aggregate's factory method. No public setters. Private/protected constructors.
- **Value Objects**: Immutable. Override `Equals`/`GetHashCode`. Use `record` types where appropriate.
- **Strongly Typed IDs**: Use typed IDs (e.g., `OrderId`, `CustomerId`) implemented as value objects — never raw primitives.
- **Repositories**: Interfaces in Domain/Application; implementations in Infrastructure. Only for aggregate roots. Use business-intent method names (`PlaceOrder`, `FindOrderByNumber`) — never CRUD names (`Get`, `Create`, `Update`, `Delete`).
- **Domain Events**: Raise for significant state changes.
- Do not use navigation properties between aggregates.

---

## C# Coding Style

- **File-scoped namespaces** for all files (e.g., `namespace MyProject.Domain.Order;`).
- **One type per file**.
- **`sealed` by default** — only mark `virtual` or leave unsealed when inheritance is intentional.
- Use `var` for local variables when the type is obvious.
- Use `nameof(param)` in exception messages — never hardcode parameter names as strings.
- No `else` keyword — use early returns and guard clauses (fail fast).
- Max one level of indentation per method; extract methods to reduce nesting.
- No abbreviations in names.
- XML doc comments (`///`) for public APIs.
- No commented-out code in commits.

### Object Calisthenics (applied to domain classes)

1. One level of indentation per method
2. No `else` keyword — use early returns
3. Wrap primitives and strings in value objects
4. First-class collections — a class with a collection attribute has no other attributes
5. One dot per line
6. No abbreviations
7. Keep entities small: max 10 methods, ~50 lines, 10 classes per namespace
8. Max two instance variables per class (loggers excluded)
9. No public getters/setters on domain classes — expose behavior, not data

---

## Testing

- **Framework**: `xunit.v3`
- **Mocks**: `FakeItEasy`
- **Integration / contract tests**: `Testcontainers`, `Microcks`

### Test Organization

- Unit tests in `tests/[Project].UnitTests/` — Domain and Application layers only.
- Integration tests in `tests/[Project].IntegrationTests/` — Infrastructure, Api, and architecture validation.
- Use **folder-per-class, class-per-method** for complex classes:

```
tests/[Project].UnitTests/
  OrderTests/
    CreateTests.cs
    PlaceOrderTests.cs
  OrderIdTests/
    CreateTests.cs
    EqualsTests.cs
```

- Suffix test folders with `Tests` (e.g., `OrderTests/`, not `Order/`) to avoid namespace collisions.
- Test names should be descriptive — describe the scenario and expected outcome. Do not use `Arrange/Act/Assert` in method names.
- Run `dotnet test` after every significant change.

---

## Commits & Branching

- **Branching**: GitHub Flow — feature branches → `main` via PR.
- **Commit messages**: [Conventional Commits](https://www.conventionalcommits.org/)

```
<type>[optional scope]: <description>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`  
Scope (optional): layer or feature (e.g., `api`, `domain`, `infrastructure`)  
First line ≤ 72 characters. Imperative, lowercase, no trailing period.

Examples:
```
feat(api): add order creation endpoint
fix(domain): correct order quantity validation
test(order): add unit tests for PlaceOrder
```
