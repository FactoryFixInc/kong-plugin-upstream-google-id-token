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

## Package APIs and Libraries

- Treat exported symbols as a public API. Flag changes to exported names, types, runtime behavior, module options, or generated artifacts that could break consuming repos.
- Keep package entry points in sync with implementation changes, especially `src/index.ts`, generated type declarations, package `main`/`types` fields, and documented import paths.
- Package code should stay reusable and consumer-agnostic. Avoid service-specific environment variables, controllers, queues, database migrations, Cloud Run assumptions, or application orchestration unless that is the package's explicit purpose.
- Do not add runtime dependencies casually. Prefer existing dependencies, keep framework dependencies compatible with consuming services, and use peer dependencies when consumers are expected to provide the framework package.
- Preserve stack traces and typed errors. Generic packages should throw normal `Error` subclasses or package-specific errors, not NestJS HTTP exceptions, unless the package is explicitly an HTTP transport helper.
- Security-sensitive packages, such as auth, guard, token, or request-context helpers, should avoid logging credentials or tokens and should include tests for authorization edge cases.
- Type-only, DTO, or interface packages should avoid adding runtime behavior unless it is already part of the package pattern.
- Utility functions that process collections should keep concurrency bounded when they perform independent async work. Prefer the repo's bounded utility, such as `promisePool()`, over serial awaits or unbounded `Promise.all()` / `Promise.allSettled()`.
- Tests should cover the package's public API, compatibility behavior, exported types where practical, error paths, and edge cases that would affect consumers.
- Use the package's existing build, lint, and test scripts. Do not suggest service-only commands, Docker service stacks, database migrations, or deployment checks for package-only changes.
