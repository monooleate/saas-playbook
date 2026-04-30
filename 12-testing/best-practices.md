# 12 — Testing — Best Practices

> **Status:** Outline.

## Sources

- Kent C. Dodds' testing trophy.
- Vitest, Playwright docs.
- *xUnit Test Patterns* (Meszaros) for the deeper why.

## Practices to expand

- Test the behaviour, not the implementation.
- Each test should fail for one reason.
- Avoid mocks for code you wrote yourself; mock external services only.
- Integration tests over unit tests when business logic lives in DB
  queries.
- E2E for the 3–5 most critical user flows.
- "Don't test types in unit tests; the type checker already does it."
