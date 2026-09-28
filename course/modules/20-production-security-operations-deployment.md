# Module 20 — Production Security, Reliability, Observability, and Deployment

> **Build goal:** prepare a production release of the Northstar SPA and its separately operated API. Produce a security checklist, CI quality gates, error/monitoring plan, deployment routing/cache policy, smoke test, and rollback notes.

## 1. Production is a trust boundary

The browser bundle is public and user-controlled. Every `VITE_*` value is visible in the built client. Do not place database credentials, signing keys, private payment keys, or confidential service secrets in frontend environment files. Secrets belong on the server/deployment secret manager.

The same-origin deployment uses a static SPA plus `/api` reverse proxy/API. The browser uses relative URLs. In development, Vite proxies the path; in production, the hosting layer routes it. CORS is not authentication or authorization.

## 2. Security review checklist

- **XSS:** render text as text; avoid unsafe HTML. If rich text is required, define sanitization. Review `dangerouslySetInnerHTML`, third-party scripts, and dependency surfaces.
- **CSP/Trusted Types:** deploy a policy appropriate to the app, test report-only before enforcement where practical, and document allowances. React's Trusted Types integration does not sanitize arbitrary HTML for you.
- **Sessions/CSRF:** secure HttpOnly cookies, HTTPS, expiry/revocation, same-site/origin/token policy, and safe logout.
- **Authorization:** API enforces per-request role, ownership, and business rules. Test IDOR and privilege escalation.
- **Input validation:** parse request/response boundaries, validate on server, and allowlist update fields.
- **Uploads:** enforce permission, size/type/content validation server-side; use short-lived scoped upload authorization and private object storage where needed.
- **Dependencies:** keep a lockfile, update intentionally, review advisories and transitive dependencies, remove unused packages.
- **Privacy:** minimize profile/order data, redact logs/telemetry, and define retention and access policy.
- **Payment:** use provider tokens/sandbox, never collect raw card data in the capstone UI.

A frontend cannot fix missing backend authorization. Include API contract tests and security review with the backend team.

## 3. Error boundaries and operational errors

Render failures, request failures, validation failures, and permission denials have different recovery paths. Use route/page error boundaries for render/loader failures, Query errors for server-data requests, and field errors for form validation. Log a request/correlation ID when available, not request bodies containing secrets.

Retry policy must match operation safety:

- Retry transient safe reads selectively with backoff and respect `Retry-After`.
- Do not blindly retry orders, payments, role changes, or other side effects.
- Use idempotency keys for retryable mutation semantics.
- Do not loop on 401, retry 403, or hide 409/412 concurrent changes.

A conservative TanStack Query default for **read queries** can retry only non-HTTP failures a small number of times. Mutations remain non-retryable unless a specific API operation has an idempotency contract:

```ts
import { QueryClient } from "@tanstack/react-query";
import { ApiError } from "./api-client";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        if (error instanceof ApiError) return false; // do not retry HTTP failures blindly
        if (!(error instanceof TypeError)) return false; // don't retry schema/programming errors
        return failureCount < 2; // bounded retry for fetch network failures
      },
    },
    mutations: { retry: false },
  },
});
```

This is a starting policy, not a universal setting: classify rate limits using `Retry-After`, distinguish permanent client/network configuration failures from transient outages, and opt into mutation retries only when the server guarantees idempotency.

## 4. Observability and supportability

Define events/metrics for route failures, API error rates, session bootstrap failure, checkout outcomes, and core vitals. Avoid logging email/password/session values, raw payment data, or full user payloads. Use request IDs to correlate browser-safe reports with server logs. Provide a user-friendly reference ID where support needs it.

Feature flags and rollout controls should be server/deployment-owned when they affect sensitive business operations. Do not use client flags to grant authority.

## 5. Build and deploy the SPA

A production checklist:

1. Run a clean lockfile install (`npm ci`) on the supported Node version.
2. Run format check, lint, typecheck, unit/component tests, browser tests, and production build.
3. Verify the output path/base URL and SPA fallback for nested route refreshes.
4. Configure CDN cache: hashed static assets can be long-lived/immutable; HTML should revalidate so it points to current assets.
5. Route `/api` to the backend and validate session-cookie domain/path/security settings.
6. Set production environment configuration without embedding server secrets.
7. Smoke-test landing, deep link, login/logout, API failure, and one authorized/forbidden flow.
8. Monitor rollout, error rates, core journeys, and Core Web Vitals; keep rollback steps ready.

`vite preview` is for local verification of the built output, not a production server. Static hosting must return the SPA entry document for client routes while preserving real asset/API 404 behavior.

## 6. CI and dependency maintenance

A CI pipeline should execute the same scripts as local development. Keep browser setup reproducible, cache only safe build artifacts, and avoid exposing secrets to untrusted pull-request code. Pin actions to trusted versions/SHAs according to the team's supply-chain policy. Review lockfile diffs and package advisories rather than blindly running a force-upgrade.

## Debugging lab

- Works at `/` but refresh at `/admin/products` returns 404: missing SPA fallback.
- Session cookie works locally but not production: check HTTPS, `Secure`, `SameSite`, host/domain, path, proxy headers, and CORS/credentials.
- Build contains a private key: remove it, rotate/revoke the leaked secret, and move it server-side; deleting it from current source alone is insufficient.
- Every 500 retries until the user leaves: adjust retry policy and expose safe recovery.
- Deploy succeeds but old JavaScript requests deleted chunks: review HTML/asset cache policy and release atomicity.

## Exercises

1. Create a release checklist with owners for browser and backend security tasks.
2. Inspect a built bundle for accidental secrets and public `VITE_*` configuration.
3. Simulate nested-route refresh on a static preview and deployment-like server.
4. Add request ID to safe error UI and verify no credentials are logged.
5. Document retry/idempotency policy for product edit, role change, cart update, and order placement.
6. Write rollback and smoke-test steps for a failed deployment.

## Summary and production tips

**Summary:** production quality spans the build, browser, API, edge, monitoring, and recovery. The frontend bundle is public; the backend owns secrets and authorization. Deployment routing and cache policy are part of correctness.

```text
review → clean install → checks/tests/build → deploy + route → smoke test → monitor → rollback if needed
```

- Never ship secrets in client variables.
- Treat authorization, session, and validation as server responsibilities.
- Give every failure a safe message and a support correlation path.
- Keep retry behavior tied to idempotency and domain semantics.
- Revisit dependency versions, docs, and advisories on a planned cadence.

## Official documentation

- [Vite static deployment](https://vite.dev/guide/static-deploy)
- [Vite environment variables](https://vite.dev/guide/env-and-mode)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP XSS Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP CSP Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [OWASP dependency management](https://cheatsheetseries.owasp.org/cheatsheets/Dependency_Management_Cheat_Sheet.html)
- [MDN Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [MDN Trusted Types](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API)
- [React Trusted Types integration announcement](https://react.dev/blog/2026/09/09/react-19-3)
- [Playwright CI](https://playwright.dev/docs/ci-intro)
- [GitHub Actions](https://docs.github.com/en/actions)
- [Backend Contract](../capstone/backend-contract.md)

## Readiness criteria

You can explain where secrets belong, describe SPA/API routing and cache behavior, differentiate safe retries from unsafe writes, handle nested route refresh, define measurable release gates, and document monitoring/rollback without treating deployment as “upload dist and hope.”
