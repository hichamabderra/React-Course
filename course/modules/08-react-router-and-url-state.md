# Module 8 — React Router 8 Data Mode and URL State

> **Build goal:** add nested storefront, account, and admin route layouts; make catalogue search/filter/sort/page state shareable in the URL; add route-level pending, error, and not-found experiences.

## 1. A URL is part of the interface

A URL is not just an address. It can represent shareable, reloadable navigation state. A product slug belongs in a path segment; catalogue search/sort/page usually belongs in search parameters; an open tooltip usually does not belong in the URL.

```text
/products?q=headphones&category=audio&sort=price.asc&page=2
```

If a user copies this URL, reloads it, or uses Back/Forward, the application should restore the corresponding catalogue view. Keep canonical parsing in one place and validate unknown/invalid values rather than casting strings blindly.

```ts
type CatalogSort = "featured" | "price.asc" | "price.desc";

function parseCatalogSearch(search: URLSearchParams) {
  const rawSort = search.get("sort");
  const sort: CatalogSort =
    rawSort === "price.asc" || rawSort === "price.desc" ? rawSort : "featured";
  const rawCategory = search.get("category");
  const category =
    rawCategory === "audio" || rawCategory === "home" || rawCategory === "office"
      ? rawCategory
      : "all";
  const pageValue = Number(search.get("page"));

  return {
    query: (search.get("q") ?? "").trim(),
    category,
    sort,
    page: Number.isInteger(pageValue) && pageValue > 0 ? pageValue : 1,
  };
}
```

This parser is a usability boundary. The API validates query parameters again on the server.

## 2. Data Mode and application routes

React Router 8.4.0 Data Mode supports route objects, nested layouts, loaders/actions, pending state, and error boundaries while leaving the Vite SPA build/deploy boundary under our control.

```tsx
import { createBrowserRouter, Outlet } from "react-router";
import { RouterProvider } from "react-router/dom";

function AppLayout() {
  return (
    <>
      <SkipLink />
      <SiteHeader />
      <Outlet />
      <SiteFooter />
    </>
  );
}

const router = createBrowserRouter([
  {
    path: "/",
    Component: AppLayout,
    children: [
      { index: true, Component: HomePage },
      { path: "products", Component: CatalogPage },
      { path: "products/:productId", Component: ProductDetailPage },
      { path: "account", Component: AccountLayout, children: [
        { index: true, Component: AccountOverviewPage },
        { path: "orders", Component: OrdersPage },
      ] },
      {
        path: "admin",
        Component: AdminLayout,
        children: [
          { index: true, Component: AdminDashboardPage },
          { path: "products", Component: AdminProductsPage },
        ],
      },
      { path: "*", Component: NotFoundPage },
    ],
  },
]);

export function Root() {
  return <RouterProvider router={router} />;
}
```

Create the router once outside the component tree; do not recreate it during renders. Import most APIs from `react-router` and the DOM `RouterProvider` from `react-router/dom` in this Vite DOM app.

Nested routes let parent layouts own shared navigation and outlet boundaries. They do not require a giant `App.tsx` with nested conditional rendering.

## 3. Search parameters and navigation

```tsx
import { useSearchParams } from "react-router";

function CatalogPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const filters = parseCatalogSearch(searchParams);

  function setQuery(query: string) {
    setSearchParams((current) => {
      const next = new URLSearchParams(current);
      if (query) next.set("q", query);
      else next.delete("q");
      next.delete("page"); // a new query starts at page one
      return next;
    });
  }

  return <CatalogControls value={filters.query} onQueryChange={setQuery} />;
}
```

Avoid updating the URL on every keystroke if it creates noisy history entries. Decide whether edits replace the current entry or push a new one, and consider debounced commits for typeahead. A URL update should not store sensitive data or credentials.

## 4. Loaders, actions, and Query ownership

A loader can orchestrate route entry and prefetch the same Query cache the component consumes. Do not fetch the same products into a loader cache and a separate independent server cache.

```tsx
const router = createBrowserRouter([
  {
    path: "/products",
    loader: ({ request }) => {
      const url = new URL(request.url);
      const filters = parseCatalogSearch(url.searchParams);
      return queryClient.ensureQueryData(catalogQueryOptions(filters));
    },
    Component: CatalogPage,
    ErrorBoundary: RouteErrorBoundary,
  },
]);
```

`ensureQueryData` can seed/reuse the Query cache; `CatalogPage` can then subscribe with the same query key. Whether to put data orchestration in a loader or let the page query directly is an architectural choice. Use loaders when route navigation needs data/error semantics; use Query directly when that is simpler. Choose one coherent owner.

`useNavigation()` exposes navigation state for pending UI. Keep the site shell usable while a child route loads. An error boundary should show a stable message and a recovery action, not dump an exception.

## 5. Protected layouts are UX only

A `UserLayout` may redirect an unauthenticated user to sign-in; an `AdminLayout` may hide admin navigation from a normal user. These checks improve experience but do not protect `/api/admin/...`. Every API operation is authenticated and authorized by the backend. A request may still return 401 or 403 after route rendering.

Use an explicit not-found route, and distinguish a missing product from a network failure. A loader redirect is navigation behavior, not a security boundary.

## Debugging lab

- Filters disappear on refresh: state was only in component state; move shareable filters to search params.
- Back button moves through every keystroke: too many history pushes; decide between `replace` and deliberate commits.
- Query cache contains two copies: loader and component use different query keys/data sources; centralize key/options.
- A user can manually call admin APIs despite a hidden nav link: expected; fix authorization on the server.
- Invalid `page=NaN` crashes the component: parse and normalize URL input at the route boundary.

## Exercises

1. Add a product route and a nested customer account route with shared layout.
2. Persist query/category/sort/page in search params; reset page when filters change.
3. Confirm Back/Forward restores catalogue state.
4. Add pending and route error UI; test an offline navigation.
5. Document which state is in path, search params, local component state, and Query cache.

## Summary and production tips

**Summary:** routes describe navigation and layout; search params encode shareable state; loaders/actions coordinate route work; Query remains the canonical server cache when selected. Frontend route checks are not authorization.

```text
path = resource identity
search params = shareable view state
component state = ephemeral UI
Query cache = remote state
server = authority
```

- Validate URL strings as untrusted input.
- Keep route components readable by extracting feature workflows, not by hiding the entire tree in a generic router utility.
- Avoid duplicating the same fetched entity across loader and Query caches.
- Make route errors recoverable and preserve the shared shell where possible.
- Use relative API paths; the browser must not hard-code localhost.

## Official documentation

- [React Router modes](https://reactrouter.com/start/modes)
- [Data Mode installation](https://reactrouter.com/start/data/installation)
- [Routing](https://reactrouter.com/start/data/routing)
- [Route Objects](https://reactrouter.com/start/data/route-object)
- [Data Loading](https://reactrouter.com/start/data/data-loading)
- [Actions](https://reactrouter.com/start/data/actions)
- [Pending UI for Data Mode](https://reactrouter.com/start/data/pending-ui)
- [createBrowserRouter](https://reactrouter.com/api/data-routers/createBrowserRouter)
- [RouterProvider](https://reactrouter.com/api/data-routers/RouterProvider)
- [useSearchParams](https://reactrouter.com/api/hooks/useSearchParams), [useNavigation](https://reactrouter.com/api/hooks/useNavigation), [useRouteError](https://reactrouter.com/api/hooks/useRouteError)
- [Updating from v7](https://reactrouter.com/upgrading/v7)

## Readiness criteria

You can create a stable router outside React, explain nested layouts, encode/restores filters in URL state, render pending/error states, distinguish loader orchestration from Query caching, and explain why client route protection cannot secure an API.
