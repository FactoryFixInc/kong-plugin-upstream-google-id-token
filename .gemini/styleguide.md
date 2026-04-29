<!-- Generated from the docs repo: docs/process/tools/gemini-code-review. -->
<!-- Edit the source docs, then run bin/render-gemini-styleguides.sh. -->

# FactoryFix Gemini Code Review Style Guide

This file is self-contained so Gemini Code Assist can review this repo without following links to companion docs.

## Review Tone

- Be pragmatic and question-first. Prefer "Have you considered...?" or "Any reason not to...?" for non-blocking feedback.
- Block only on correctness, security, data integrity, broken behavior, or missing tests for changed critical flows.
- Praise useful patterns when they are present.
- Avoid style-only comments unless the issue creates real confusion or violates an established repo convention.
- Prefer a small number of high-signal comments over many low-value nits.

## Kong Plugin Development

- These repos are Lua Kong plugin packages, not NestJS services, frontend apps, or generic TypeScript packages. Review Kong/OpenResty runtime behavior, plugin schema, rockspec packaging, and API Gateway compatibility first.
- Treat plugin name, phase handlers, `PRIORITY`, `VERSION`, schema fields, defaults, required settings, protocol support, and entity checks as public API. Changes can break Kong declarative config, service-generated OpenAPI annotations, or the API Gateway Docker image.
- Keep rockspec names and versions, plugin source paths under `kong/plugins/<plugin-name>`, README installation docs, test fixtures, and any API Gateway install scripts or pinned tags in sync.
- Be careful with plugin execution order. Priority changes can alter authentication, authorization, path rewriting, and upstream request mutation when multiple plugins run on the same request.
- Prefer Kong APIs such as `kong.request`, `kong.service.request`, `kong.response`, `kong.client`, and `kong.log` over direct `ngx` access when the Kong API supports the operation. Direct `ngx` usage should be intentional and compatible with the relevant phase.
- Do not log secrets, bearer tokens, ID tokens, service account keys, client secrets, userinfo payloads, or other sensitive headers. Debug logs in plugins can become gateway-wide operational exposure.
- Schema changes should be backward-compatible unless the rollout plan updates every Kong config that uses the plugin. Flag changed defaults, renamed config fields, relaxed auth behavior, or widened protocol applicability.
- Runtime changes should preserve Kong/OpenResty compatibility for the supported Kong versions. Avoid dependencies, Lua APIs, or Docker assumptions that the plugin test environment and API Gateway image do not provide.
- Path, header, credential, cache, auth, and response-code behavior should have focused plugin tests when changed. Use the repo's Docker, Pongo, Makefile, or existing test scripts rather than service-only commands.

## kong-plugin-upstream-google-id-token Notes

- This plugin injects a Google ID token for upstream Cloud Run service-to-service auth. Preserve the default header `X-Serverless-Authorization` so end-user `Authorization` headers remain separate.
- Audience calculation is part of the security contract. It currently derives the target audience from Kong's routed service protocol and host; changes should be tested against Cloud Run IAM expectations.
- Preserve both credential modes: metadata server token retrieval in GCP and service-account-key based token retrieval via `GOOGLE_APPLICATION_CREDENTIALS` for non-GCP environments.
- Keep token caching bounded by Google ID token lifetime. Changes to `id_token_cache_ttl`, cache keys, or refresh behavior can cause auth failures, excess metadata/API calls, or wrong-audience token reuse.
- Do not log ID tokens, signed JWT assertions, service account private keys, service account JSON content, or upstream auth headers.
- Verification should use Pongo tests for schema and token injection behavior, plus an API Gateway image/build check when packaging, rockspec, or dependency changes affect installation.
