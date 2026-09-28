# Module 14 — Admin Tables, Server Pagination, and Resilient UI States

> **Build goal:** build a responsive admin product/user data grid with server-side search, filtering, sorting, and pagination. Use TanStack Table 9.2.4 only for the complex grid; use semantic HTML for rendering. Add explicit pending, empty, error, conflict, and success states.

## 1. A table is a view over a data contract

For a small static list, a native `<table>` is enough. A production admin grid may need column sorting, selection, server pagination, filtering, and visibility. TanStack Table is headless: it computes state and row models; your app owns markup, styles, keyboard behavior, labels, and accessibility review.

Server-side sorting/filtering/pagination is often required for large or permission-scoped datasets. Don't download every user and filter in the browser; that is a privacy, performance, and authorization bug. Keep view state in the URL when it is shareable and use the same values in the Query key.

## 2. TanStack Table v9 feature registration

The course pins TanStack Table v9.2.4. Version 9 uses `useTable`, explicit features, and a redesigned state model. Do not copy v8 `useReactTable` snippets without intentionally using the compatibility path.

```tsx
import {
  createSortedRowModel,
  rowSortingFeature,
  sortFns,
  tableFeatures,
  useTable,
} from "@tanstack/react-table";

const features = tableFeatures({
  rowSortingFeature,
  sortedRowModel: createSortedRowModel(),
  sortFns,
});

function ProductTable({ columns, rows }: Props) {
  const table = useTable({ features, columns, data: rows });
  // Render the instance using semantic <table>, <thead>, <tbody>, <th>, and <td>.
  return <ProductTableMarkup table={table} />;
}
```

Register only features used. The v9 docs show rendering headers/cells with `table.FlexRender`; follow the exact installed-version examples for controlled state and row-model configuration.

For server-side data, the table should send sort/page/filter changes upward rather than sorting a partial page as if it were the full dataset. Each request input belongs in the route URL and query key; server results include current page, total, and next-page information.

## 3. UI states are distinct

A robust list should represent:

- Initial loading: skeleton or measured row placeholders with the table's accessible structure in mind.
- Background refresh: retain existing data where safe, set `aria-busy`, and announce a concise refresh status.
- Empty dataset: no records exist for this account/filter; offer a relevant action.
- No matches: clear search/filter option, not a generic error.
- Error: safe message, retry for safe reads, correlation ID for support.
- Mutation pending: prevent duplicate action or show per-row pending status.
- Conflict/precondition failure: tell the admin data changed and offer reload/review.
- Success: announce that the operation completed and reflect server response.

Do not use one global `isLoading` boolean for every route/data source. Query's pending and background fetching states answer different questions.

## 4. Responsive data tables

At small viewports, choose a deliberate strategy: a compact essential-column table, an alternative card/list layout, a horizontally scrollable region with clear affordance, or a responsive table that hides lower-priority columns. Avoid making all content inaccessible behind unexplained sideways scrolling.

For sorting controls, use actual buttons in header cells and expose current sort direction. If the UI is a normal data table with buttons, do not add `role="grid"`; grid semantics create extra keyboard obligations. Use a true composite grid only when spreadsheet-like cell navigation is needed.

## 5. Concurrent edits and safe mutations

Use ETags/`If-Match` when the backend supports optimistic concurrency. A `412` means the edit is stale; do not overwrite silently. A delete/archive action should state what will happen, require confirmation when destructive, and expose a cancel path. Server authorization and audit policy remain authoritative.

## Debugging lab

- Every page shows the same first 20 records: pagination state isn't reaching the server/query key.
- Price sort only sorts current page: sort is applied client-side to a partial server page.
- Search shows 0 while request is still pending: stale count/loading state mixed with current filter.
- An optimistic delete removes the row permanently after a 412: rollback/reload strategy is missing.
- Table has `role="grid"` but arrows/tab don't work as specified: semantics exceed actual keyboard behavior.

## Exercises

1. Build the product grid with server search/sort/page from the API contract.
2. Make URL Back/Forward restore table filters and page.
3. Add an empty-state action and separate no-match message.
4. Simulate a stale ETag; require reload/review before saving.
5. Test the admin list at mobile, tablet, desktop, 200% zoom, and keyboard-only.

## Summary and production tips

**Summary:** table logic and table semantics are different responsibilities. TanStack Table v9 supplies a headless state/model; the application renders accessible markup and coordinates server query state.

```text
URL/table controls → API query key → server-scoped page → semantic table → explicit state feedback
```

- Use server pagination for large or protected lists.
- Do not virtualize until real DOM volume is measured.
- Preserve stable row identity; never key rows by current page index.
- Use a normal table unless a composite grid's richer keyboard model is needed.
- Keep row actions permission-aware in UX and re-authorized by the server.

## Official documentation

- [TanStack Table v9 React overview](https://tanstack.com/table/latest/docs/framework/react/overview)
- [TanStack Table v9 Quick Start](https://tanstack.com/table/latest/docs/framework/react/quick-start)
- [Migration to v9](https://tanstack.com/table/latest/docs/framework/react/guide/migrating)
- [Table state](https://tanstack.com/table/latest/docs/framework/react/guide/table-state)
- [Sorting](https://tanstack.com/table/latest/docs/framework/react/guide/sorting)
- [Column filtering](https://tanstack.com/table/latest/docs/framework/react/guide/column-filtering)
- [Pagination](https://tanstack.com/table/latest/docs/framework/react/guide/pagination)
- [TanStack Query background fetching](https://tanstack.com/query/latest/docs/framework/react/guides/background-fetching-indicators)
- [WAI-ARIA table pattern](https://www.w3.org/WAI/ARIA/apg/patterns/table/)
- [TanStack Virtual](https://tanstack.com/virtual/latest/docs/framework/react/introduction)

## Readiness criteria

You can explain why v9 uses explicit features, keep server filters/sort/page in request identity, render distinct states, handle conflicts, and choose a mobile table strategy that preserves task completion without accidental grid semantics.
