# Module 17 — Testing Strategy: Vitest, Testing Library, MSW, and Playwright

> **Build goal:** create fast unit/component tests, realistic HTTP integration tests, and a small browser suite for login, search, authorization, admin CRUD, and checkout retry behavior. Use Vitest 5.0.2, Testing Library, MSW 2.15.0, and Playwright 1.63.0 from the verified stack.

## 1. Tests are layered evidence

A single test level cannot provide every kind of confidence.

| Layer | Best for | Example |
|---|---|---|
| Pure unit | Deterministic domain logic | Parse filters, format money, compute cart merge preview. |
| Component/integration | User-visible interaction and state | Search input updates list; 422 appears on field. |
| Network contract | Request/response behavior and failures | MSW returns 403/empty/conflict; API client maps it. |
| End-to-end | Critical browser journey | Sign in, search, add to cart, submit safe order. |

Test public outcomes rather than internal Hook calls or implementation details. A refactor should not break tests if behavior remains equivalent.

## 2. Vitest configuration

Vitest uses Vite config and transforms, so configuration stays close to the app. React Testing Library needs a DOM-like environment. This course uses **jsdom 30.1.0** for component tests; jsdom is not a real browser, so Playwright remains necessary. The explicit `https://northstar.test/` URL is only the simulated test origin: it lets the app's same-origin `/api` client build absolute requests for MSW to intercept. It is not a production API host.

```ts
// vitest.config.ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    environmentOptions: {
      jsdom: { url: "https://northstar.test/" },
    },
    setupFiles: ["./src/test/setup.ts", "./src/test/msw-lifecycle.ts"],
    restoreMocks: true,
    clearMocks: true,
  },
});
```

Install the test dependencies at the verified pins:

```bash
npm install --save-dev --save-exact \
  vitest@5.0.2 @vitest/coverage-v8@5.0.2 jsdom@30.1.0 \
  @testing-library/react@16.3.3 @testing-library/user-event@14.6.7 \
  @testing-library/dom@10.4.2 @testing-library/jest-dom@7.0.1 \
  msw@2.15.0 @playwright/test@1.63.0
```

This installs the course's test layer in one go; a real team may add Playwright's browser binaries in CI setup and should update packages intentionally. Include a test script such as `"test": "vitest run"` and a watch script such as `"test:watch": "vitest"`.

```ts
// src/test/setup.ts
import "@testing-library/jest-dom/vitest";
import { cleanup } from "@testing-library/react";
import { afterEach } from "vitest";

afterEach(() => cleanup());
```

Tests import `describe`, `it`, `expect`, and `vi` from Vitest rather than relying on implicit globals. Isolated imports make dependencies obvious.

## 3. Testing Library: use the interface as a person does

```tsx
import { useState } from "react";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { expect, it, vi } from "vitest";

it("reports a changed search value", async () => {
  const user = userEvent.setup();
  const onValueChange = vi.fn();

  function SearchHarness() {
    const [value, setValue] = useState("");
    return (
      <SearchBox
        value={value}
        onValueChange={(next) => {
          onValueChange(next);
          setValue(next);
        }}
      />
    );
  }

  render(<SearchHarness />);
  await user.type(screen.getByRole("searchbox", { name: "Search products" }), "lamp");

  expect(onValueChange).toHaveBeenLastCalledWith("lamp");
  expect(screen.getByRole("searchbox", { name: "Search products" })).toHaveValue("lamp");
});
```

Prefer role/name/label queries. `getByTestId` can be useful when no user-facing contract exists, but a test ID should not replace a missing accessible name. Use `findBy...` for async appearance and `waitFor` for a condition that becomes true; do not sleep arbitrary milliseconds.

## 4. MSW: mock HTTP, not components

MSW intercepts the request at the network boundary, so tests still execute the real API client and Query function.

```ts
// src/test/server.ts
import { setupServer } from "msw/node";

export const server = setupServer();
```

```ts
// src/test/msw-lifecycle.ts (or setup.ts)
import { afterAll, afterEach, beforeAll } from "vitest";
import { server } from "./server";

beforeAll(() => server.listen({ onUnhandledRequest: "error" }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

Example one-test override using MSW 2 APIs:

```ts
import { http, HttpResponse } from "msw";
import { server } from "../test/server";

server.use(
  http.get("/api/products", () =>
    HttpResponse.json({ data: [], meta: { page: 1, pageSize: 24, total: 0, hasNextPage: false } }),
  ),
);
```

Model success, empty, invalid input, 401, 403, 409/412, 422, 429, 500, malformed JSON, and network failure. Keep fixtures deterministic. Fail on unhandled requests so a test cannot pass while silently skipping the expected API call.

## 5. TanStack Query test isolation

Each test should get a fresh `QueryClient`; disable retries for deterministic error tests and clear cache by isolation rather than sharing a global stale client.

```tsx
function createTestQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });
}
```

A wrapper can provide `QueryClientProvider` to a test render. Do not assert private cache implementation unless cache behavior itself is the contract; prefer visible loading/error/data outcomes.

## 6. Playwright: real browser journeys

Playwright 1.63.0 runs browser tests with locators, assertions, network routing, traces, and multiple browser projects. Configure `use.baseURL` in `playwright.config.ts` to point at the locally served production build or a controlled test environment; relative `page.goto` calls require that base URL. Example:

```ts
import { expect, test } from "@playwright/test";

test("search results are reflected in the URL", async ({ page }) => {
  await page.route("**/api/products?**", async (route) => {
    await route.fulfill({
      status: 200,
      contentType: "application/json",
      body: JSON.stringify({ data: [], meta: { page: 1, pageSize: 24, total: 0, hasNextPage: false } }),
    });
  });

  await page.goto("/products");
  await page.getByRole("searchbox", { name: "Search products" }).fill("headphones");
  await expect(page).toHaveURL(/q=headphones/);
  await expect(page.getByRole("heading", { name: "No products match" })).toBeVisible();
});
```

Use Playwright route mocks for isolated browser behavior or MSW in the browser if the project deliberately configures its service worker. Do not layer two interceptors without understanding which one receives a request. Test critical behavior against a staging API only where environment stability and test-data lifecycle are controlled.

## 7. Flakiness and debugging

- Prefer locator actions and auto-retrying web assertions over fixed `waitForTimeout`.
- Use traces/screenshots/video on CI retries for diagnosis; redact sensitive test data.
- Avoid tests depending on current time, global test ordering, random fixtures, or external production APIs.
- Use deterministic clocks/IDs when necessary, then reset them.
- Separate product defects from test setup defects; a broad mock can accidentally hide a bad request.
- Keep a test close to the behavior owner while protecting public contracts.

## Suggested capstone test matrix

- Pure: search parsing, price minor-unit conversion, guest-cart merge validation.
- Component: field error associations, menu/dialog keyboard behavior, empty/loading/error states.
- HTTP/Query: key changes, cancellation, 401/403, rollback, conflict, retries.
- Playwright: sign-in refresh, catalogue URL restoration, order idempotency response, admin-denied access, mobile layout.

## Exercises

1. Write a pure test for catalogue sorting without source mutation.
2. Test search by accessible role/name and assert a user-visible result.
3. Add MSW handlers for 422 and 403; verify field error versus permission error.
4. Test Query retry behavior without waiting through backoff delays.
5. Add a Playwright path through search and a mocked cart response; run it on desktop and a mobile device profile.
6. Intentionally add a fixed timeout, observe flakiness, then replace it with a web assertion.

## Summary and production tips

**Summary:** Vitest runs unit/component tests; Testing Library verifies interaction; MSW provides HTTP behavior; Playwright covers full-browser journeys. A test is useful when it catches a meaningful regression at reasonable maintenance cost.

```text
pure rules → component behavior → HTTP integration → critical browser journey
```

- Test errors and empty states, not only success.
- Keep mocks aligned with the backend contract.
- Don't assert implementation details just because they are easy to access.
- Run lint/typecheck/build/tests in CI; use trace artifacts to debug, not to excuse flaky tests.
- Accessibility scans are one layer; include manual keyboard checks.

## Official documentation

- [Vitest Guide](https://vitest.dev/guide/), [mocking](https://vitest.dev/guide/mocking), [projects](https://vitest.dev/guide/projects), and [environment options](https://vitest.dev/config/environmentoptions)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Testing Library queries](https://testing-library.com/docs/queries/about/)
- [user-event](https://testing-library.com/docs/user-event/intro/)
- [MSW](https://mswjs.io/docs/) and [MSW 2.0 release](https://mswjs.io/blog/introducing-msw-2.0/)
- [TanStack Query testing](https://tanstack.com/query/latest/docs/framework/react/guides/testing)
- [Playwright Test](https://playwright.dev/docs/intro)
- [Playwright best practices](https://playwright.dev/docs/best-practices)
- [Playwright network](https://playwright.dev/docs/network)
- [Playwright trace viewer](https://playwright.dev/docs/trace-viewer-intro)
- [jsdom package](https://www.npmjs.com/package/jsdom)

## Readiness criteria

You can choose the right test layer, query the UI by accessible semantics, isolate Query state, model HTTP failures with MSW, write a non-flaky Playwright journey, and explain what jsdom cannot tell you about a real browser.
