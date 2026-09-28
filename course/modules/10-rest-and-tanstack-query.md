# Module 10 — REST Contracts, API Client, and TanStack Query v5

> **Build goal:** replace the static catalogue fixture with `GET /api/products`; add search/filter/page caching, product detail, admin product mutations, loading/error/empty states, cancellation, and one carefully chosen optimistic interaction. Use the contract in [Backend Contract](../capstone/backend-contract.md).

## 1. HTTP is a contract, not a function call

A browser request has a method, URL, headers, body, credentials policy, cancellation signal, status, response headers, and body. A successful network connection does not mean the HTTP operation succeeded. `fetch` resolves for 401/404/500; check `response.ok` and parse errors explicitly.

The API contract defines stable request and response shapes, pagination rules, money, error codes, authorization, and retry/idempotency semantics. A TypeScript interface alone cannot prove JSON matches that shape. Parse external data at runtime at a trust boundary; Module 13 develops Zod schemas.

## 2. A small typed transport boundary

```ts
type FieldIssue = { path: readonly string[]; message: string };
type ApiProblem = {
  title: string;
  code: string;
  requestId?: string;
  errors: readonly FieldIssue[];
};

function isRecord(value: unknown): value is Record<string, unknown> {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}

function parseProblem(value: unknown): ApiProblem {
  if (!isRecord(value)) {
    return { title: "The request could not be completed.", code: "UNKNOWN_ERROR", errors: [] };
  }

  const errors = Array.isArray(value.errors)
    ? value.errors.flatMap((item): FieldIssue[] => {
        if (!isRecord(item) || !Array.isArray(item.path) || typeof item.message !== "string") return [];
        const path = item.path.filter((part): part is string => typeof part === "string");
        return [{ path, message: item.message }];
      })
    : [];

  return {
    title: typeof value.title === "string" ? value.title : "The request could not be completed.",
    code: typeof value.code === "string" ? value.code : "UNKNOWN_ERROR",
    requestId: typeof value.requestId === "string" ? value.requestId : undefined,
    errors,
  };
}

async function readProblemSafely(response: Response): Promise<ApiProblem> {
  try {
    const body: unknown = await response.json();
    return parseProblem(body);
  } catch {
    return { title: "The request could not be completed.", code: "UNKNOWN_ERROR", errors: [] };
  }
}

export class ApiError extends Error {
  constructor(
    message: string,
    readonly status: number,
    readonly code: string,
    readonly requestId?: string,
    readonly fieldErrors: readonly FieldIssue[] = [],
  ) {
    super(message);
    this.name = "ApiError";
  }
}

export async function requestJson<T>(
  path: string,
  decode: (payload: unknown) => T,
  init: RequestInit = {},
): Promise<T> {
  const headers = new Headers(init.headers);
  headers.set("Accept", "application/json");
  const isFormData =
    typeof FormData !== "undefined" && init.body instanceof FormData;
  if (init.body && !isFormData && !headers.has("Content-Type")) {
    headers.set("Content-Type", "application/json");
  }

  const url = new URL(`/api${path}`, window.location.origin);
  const response = await fetch(url, {
    ...init,
    credentials: "include",
    headers,
  });

  if (!response.ok) {
    const problem = await readProblemSafely(response);
    throw new ApiError(
      problem.title ?? "The request could not be completed.",
      response.status,
      problem.code,
      problem.requestId,
      problem.errors,
    );
  }

  const payload: unknown = await response.json();
  return decode(payload);
}
```

The caller supplies a runtime decoder: Module 13 uses Zod schemas, while a narrow hand-written parser can be used before that dependency is introduced. Assigning JSON to `unknown` prevents an implicit `any` from being treated as trusted. Callers must `JSON.stringify` JSON request bodies; this helper sets headers but does not serialize arbitrary objects. Keep 204/no-content responses in a separate helper with an explicit `Promise<void>` return rather than pretending they are arbitrary `T`. Don't apply JSON `Content-Type` to `FormData`; the browser supplies the multipart boundary.

`credentials: "include"` is required for cookie-backed requests in this contract. The URL is constructed from the browser's current origin plus the relative `/api` path; no API host or localhost address is hard-coded. The browser stays same-origin and deployment proxy configuration remains outside the client bundle. Never read or write the HttpOnly session cookie from JavaScript.

## 3. Query keys model remote identity

TanStack Query v5 uses object options. A query key uniquely represents the request inputs that change its result.

```ts
export const productKeys = {
  all: ["products"] as const,
  lists: () => [...productKeys.all, "list"] as const,
  list: (filters: CatalogFilters) => [...productKeys.lists(), filters] as const,
  detail: (id: string) => [...productKeys.all, "detail", id] as const,
};

function useProducts(filters: CatalogFilters) {
  return useQuery({
    queryKey: productKeys.list(filters),
    queryFn: ({ signal }) =>
      requestCatalogProducts(filters, signal),
    staleTime: 30_000,
  });
}
```

If the query function reads a filter, that filter must be in the key. Otherwise the cache can return data from another search. Treat keys like a serializable request identity, not an arbitrary label. Don't put secrets or full private user data into the key.

The provided `AbortSignal` should reach `fetch`:

```ts
function requestCatalogProducts(
  filters: CatalogFilters,
  signal: AbortSignal,
): Promise<CatalogPageData> {
  const params = new URLSearchParams(serializeFilters(filters));
  return requestJson(
    `/products?${params.toString()}`,
    parseCatalogPage,
    { signal },
  );
}
```

TanStack Query may keep unused results in cache by design. Understand `staleTime`, `gcTime`, focus refetch, retry behavior, and invalidation before changing defaults globally.

## 4. Loading, background fetch, empty, and error are different

```tsx
function CatalogResults({ filters }: { filters: CatalogFilters }) {
  const query = useProducts(filters);

  if (query.isPending) return <CatalogSkeleton />;
  if (query.isError) return <RequestError error={query.error} onRetry={query.refetch} />;
  if (query.data.data.length === 0) return <CatalogEmpty onClear={clearFilters} />;

  return (
    <section aria-busy={query.isFetching}>
      {query.isFetching && <p role="status">Refreshing products…</p>}
      <ProductGrid products={query.data.data} />
    </section>
  );
}
```

A background refetch with existing data should not necessarily replace the whole screen with a spinner. An empty success is not an error. A 401 should enter session recovery; 403 should not repeatedly refresh. Rate limits should respect `Retry-After` and not trigger aggressive loops.

## 5. Mutations and cache coherence

Use the server response to update a detail cache when safe; invalidate list keys when filters/sort/total counts could change. Avoid optimistic editing a complex paginated table if rollback/conflict behavior is unclear.

For a cart quantity update, Query v5's `onMutate` context can snapshot and restore a cache value:

```tsx
const updateQuantity = useMutation({
  mutationFn: ({ productId, quantity }: Variables) =>
    api.put(`/cart/items/${productId}`, { quantity }),
  onMutate: async ({ productId, quantity }) => {
    await queryClient.cancelQueries({ queryKey: ["cart"] });
    const previous = queryClient.getQueryData<Cart>(["cart"]);
    queryClient.setQueryData<Cart>(["cart"], (cart) =>
      cart && updateLineQuantity(cart, productId, quantity),
    );
    return { previous };
  },
  onError: (_error, _variables, context) => {
    if (context?.previous) queryClient.setQueryData(["cart"], context.previous);
  },
  onSettled: () => queryClient.invalidateQueries({ queryKey: ["cart"] }),
});
```

This assumes cart data is cached under `['cart']` and `updateLineQuantity` is pure. A production key factory should name it. If the mutation can race with another update, snapshot rollback may overwrite newer work; serialize, use operation IDs, use server versioning, or avoid cache-level optimism.

## 6. Pagination and prefetching

For offset pagination, include `page` and `pageSize` in the query key. Use placeholder previous data only when the UI clearly indicates which page is currently displayed. The server owns total count and permission-scoped records. For very large/changing datasets, cursor pagination may be more stable.

Prefetch the next page or product detail only where the expected user journey benefits; prefetching costs bandwidth and can retrieve data the user never views.

## Debugging lab

- Search results from `q=lamp` appear under `q=chair`: search term missing from key.
- Cancel does not stop network activity: query function did not pass its signal to `fetch`.
- A 500 renders as successful JSON: client failed to check `response.ok`.
- Mutation retries create duplicate orders: unsafe side effect lacks idempotency; do not blindly enable retries.
- Optimistic item disappears after a conflict: reconcile/refetch from authoritative response and explain the conflict.

## Exercises

1. Implement list/detail query keys and test that changing every filter produces a distinct key.
2. Add an error component that distinguishes authentication, forbidden, validation, rate-limit, and server errors without showing stack traces.
3. Add admin product create/edit/delete mutations and choose update-versus-invalidate per query.
4. Add pagination and a retry policy that does not retry 4xx/domain errors.
5. Add an MSW test for empty results, invalid JSON, 401, 403, 422, 429, and 500.

## Summary, production tips, and mental model

**Summary:** the API client owns transport/error normalization; runtime schemas own boundary validation; Query owns remote cache lifecycle; the server owns durable truth.

```text
route/UI → query key → query function → relative /api fetch → status/error parse → runtime schema → Query cache → UI state
```

- Keep one authoritative server-data cache.
- Treat Query defaults as product behavior, not magic.
- Make query keys reflect all input dependencies.
- Forward `AbortSignal`; cancel reads, not assumptions about server-side mutation rollback.
- Prefer simpler optimistic UI over complex cache surgery unless multiple surfaces need immediate consistency.
- Retry safe reads deliberately; use idempotency for retryable writes.

## Official documentation

- [TanStack Query React overview](https://tanstack.com/query/latest/docs/framework/react/overview)
- [Important defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)
- [Queries](https://tanstack.com/query/latest/docs/framework/react/guides/queries) and [query keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys)
- [Mutations](https://tanstack.com/query/latest/docs/framework/react/guides/mutations)
- [Query invalidation](https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation)
- [Optimistic updates](https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates)
- [Query cancellation](https://tanstack.com/query/latest/docs/framework/react/guides/query-cancellation)
- [Paginated queries](https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries)
- [Testing](https://tanstack.com/query/latest/docs/framework/react/guides/testing)
- [TanStack Query v5 migration](https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5)
- [MDN Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
- [MSW documentation](https://mswjs.io/docs/)

## Readiness criteria

You can trace a request from UI to backend and back; distinguish status from network failure; design stable keys; handle pending/error/empty/background refresh; forward cancellation; choose invalidation/update/optimism; and explain why client types do not validate JSON.
