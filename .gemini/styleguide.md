<!-- Generated from the docs repo: docs/process/tools/gemini-code-review. -->
<!-- Edit the source docs, then run bin/render-gemini-styleguides.sh. -->

# FactoryFix Gemini Code Review Style Guide

This file is self-contained so Gemini Code Assist can review this repo without following links to companion docs.

## Release PRs

If this PR targets the `prod` branch, do not perform a full code review. FactoryFix `staging` to `prod` pull requests are release PRs where code has already been reviewed before it reached `staging`.

For `prod` PRs, post a short, creative deployment blessing or well-wish instead of review comments. Keep it brief and different each time.

## Review Tone

- Be pragmatic and question-first. Prefer "Have you considered...?" or "Any reason not to...?" for non-blocking feedback.
- Block only on correctness, security, data integrity, broken behavior, or missing tests for changed critical flows.
- Praise useful patterns when they are present.
- Avoid style-only comments unless the issue creates real confusion or violates an established repo convention.
- Prefer a small number of high-signal comments over many low-value nits.

## Backend Architecture

- Keep clean architecture boundaries intact: controllers handle transport concerns, use cases own business logic, and persistence/adapters only fetch, store, or send data.
- Controllers should translate HTTP or messaging inputs into use case calls. They should not contain business rules, persistence calls, or cross-service orchestration.
- Use cases should expose a single clear `execute()` entry point for a business flow. Split separate flows into separate use cases when branching logic stops being trivial.
- Persistence classes should not validate business rules, throw HTTP exceptions, or decide what missing data means. Return data or `null` and let the caller decide.
- Do not add DAO interfaces, wrapper services, or abstractions unless they remove real complexity or match an existing local pattern.

## Error Handling and Logging

- Preserve stack traces. Pass real `Error` objects through error paths instead of converting them to strings.
- Use `logger.error(error)` only when the error is handled and will not be rethrown. This creates Google Cloud Error Reporting events.
- Use `logger.warn()` when adding context before throwing or rethrowing. Avoid duplicate Error Reporting events for the same failure.
- Do not log errors with interpolated strings such as ``logger.error(`message: ${error}`)`` because that loses stack trace details.
- Persistence methods that catch storage or API errors should wrap them in the repo's established persistence error type when one exists.
- Use cases should not throw NestJS HTTP exceptions such as `BadRequestException`, `NotFoundException`, or `InternalServerErrorException`.
- Domain/use case code should throw domain errors that extend `Error`. Controllers are responsible for mapping domain errors to HTTP exceptions.
- Avoid `console.log`, `console.warn`, and `console.error` in application code. Use the repo's logger.

## Security and Authorization

- Service-to-service calls should use FactoryFix service auth helpers such as `@factoryfixinc/nest-auth`. Do not forward user tokens between services.
- Validate ownership and authorization at the boundary before acting on employer IDs, user IDs, project IDs, application IDs, subscription IDs, or organization IDs.
- Protected endpoints should have the repo's established auth guards. Flag missing guards on new controllers or routes.
- Check for insecure direct object reference risks when a caller can provide an entity ID.
- Do not introduce secrets, tokens, credentials, or private keys in source files, docs, tests, Terraform, or example configs.
- Infrastructure changes should use least-privilege IAM roles and Secret Manager references for sensitive values.

## Testing

- New controllers, use cases, and critical branches should have focused tests.
- Prefer testing domain/use case behavior over thin persistence or adapter wrappers.
- Co-locate Jest unit specs next to the source files when that is the local pattern.
- Use repo scripts such as `yarn test`, `yarn lint`, `yarn build`, or `./bin/run-tests.sh`; do not suggest ad-hoc `npx` commands.
- For Docker-only repos, expect commands to run through Docker Compose or the repo's bootstrap/test scripts.
- Tests should cover happy paths, meaningful error paths, and edge cases such as missing data, zero values, authorization failures, and duplicate requests.
- Avoid tests that only assert mocks were called without proving user-visible or domain behavior.

## Data, Performance, and Operations

- New database queries should have appropriate migrations and indexes. Flag large-table filters that lack index coverage.
- Avoid loading unbounded datasets into memory. Prefer pagination, batching, streaming, or query limits.
- Avoid N+1 access patterns and repeated updates to the same entity when a single update would work.
- Prefer `promisePool()` for independent async work over collections. It is better than serially awaiting independent operations and safer than `Promise.all()` or `Promise.allSettled()` over unbounded arrays because it keeps concurrency explicit.
- Choose a concurrency limit based on downstream database, API, queue, or rate-limit constraints. Name the limit with a clear constant when the value is not obvious.
- If all-settled behavior is required, use the repo's bounded settled-pool utility when available instead of unbounded `Promise.allSettled()`.
- Firestore multi-document or read-modify-write flows should use transactions when partial updates would create inconsistent state.
- Pub/Sub topics, Cloud Tasks queues, and environment variables must be configured in all required places: service config, local Docker/test config, and infrastructure config.
- Alerting and monitoring changes should include actionable runbook context where the local infrastructure pattern supports it.
