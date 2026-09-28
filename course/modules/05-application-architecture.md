# Module 5 — Application Architecture and Component Engineering

> **Build goal:** organize Northstar into feature boundaries with explicit dependency direction. Do not create empty folders for hypothetical future work. Create the boundaries the catalogue, account, cart, and admin features actually need.

## 1. Architecture is about ownership and change

Architecture is the set of decisions that make future changes either easy or expensive. A folder tree is a map, not architecture by itself. Good boundaries answer:

- Which feature owns this behavior and its domain concepts?
- Which modules may depend on which others?
- Where do side effects enter and where do errors become user-facing?
- Which parts can be tested without starting the whole app?

A useful dependency direction for this capstone:

```text
route/page composition
  ↓
feature workflows → domain types/pure rules
  ↓                    ↑
shared UI primitives   API client/contracts
```

Features may compose shared UI and domain helpers. Shared UI should not import a page or feature implementation. API code should not import a React component. Keep the graph directed to avoid cycles.

## 2. Grow from the actual feature set

Module 1's small structure is appropriate for one screen. As the app grows, a feature-oriented layout can evolve into:

```text
src/
  app/                 # application root, providers, top-level error handling
  routes/              # route definitions and route-level composition
  shared/
    api/               # HTTP client and shared transport/error primitives
    ui/                # genuinely shared controls and layout primitives
    lib/               # small cross-feature pure utilities
  features/
    catalog/           # product views, filters, catalogue queries
    auth/              # sign-in/register/session UI and hooks
    cart/              # cart UI, client draft, server mutations
    account/           # profile/settings/orders
    admin-products/    # admin-specific product workflows
    admin-users/       # user search/role workflow
```

Do not put every type into a universal `types/` folder or every callback into `utils/`. Keep a type close to its owner until a second, real feature needs it. Avoid the “shared” dumping ground: a module is shared only if it has multiple consumers and stable semantics.

A small feature API might export only what other parts need:

```ts
// features/catalog/index.ts
export { CatalogPage } from "./CatalogPage";
export type { Product, ProductId } from "./model";
```

Barrel files are optional. Broad barrels can create circular imports, expose internals accidentally, and inhibit code splitting. Use direct imports until a stable public boundary is valuable.

## 3. Component APIs: compose before abstracting

A good component has a clear job, a small surface, and predictable behavior. Composition is often better than a large component with dozens of boolean props.

```tsx
import type { ReactNode } from "react";

interface PageHeaderProps {
  title: string;
  description?: string;
  actions?: ReactNode;
}

function PageHeader({ title, description, actions }: PageHeaderProps) {
  return (
    <header className="page-header">
      <div>
        <h1>{title}</h1>
        {description && <p>{description}</p>}
      </div>
      {actions && <div className="page-header__actions">{actions}</div>}
    </header>
  );
}
```

This component owns layout, not the product edit workflow. It accepts already meaningful children/actions and does not fetch or mutate server data.

Anti-pattern:

```tsx
<UniversalCard isProduct isAdmin isLoading hasFooter hasHeader dense compact ... />
```

When conditions interact, the abstraction hides more than it shares. Use composition, explicit variants, or separate components where behavior differs. Extract when there is a repeated stable concept—not merely because two blocks both contain a `<div>`.

## 4. Server/client boundaries are architectural boundaries

A feature may import an API function, but should not reconstruct HTTP semantics in every button. A page composes a workflow; a query hook owns server-data subscription; a presentational component receives data and callbacks.

```tsx
function ProductRow({ product, onEdit }: {
  product: Product;
  onEdit: (productId: string) => void;
}) {
  return (
    <div>
      <span>{product.name}</span>
      <button type="button" onClick={() => onEdit(product.id)}>Edit {product.name}</button>
    </div>
  );
}
```

The row does not know the HTTP verb or permission policy. The parent can decide whether to render the action for UX; the backend still authorizes the request.

## 5. Context, custom hooks, and dependency injection

Context is useful for values consumed deeply across a stable subtree (theme, localization, router/query providers). Every context update can affect its consumers, so do not use one high-churn context as a replacement for a cache or fine-grained store. A custom Hook shares behavior; it does not share state unless it reads the same external store/context.

For tests, prefer simple injection at the boundary (a provider, MSW HTTP handler, or function parameter) over building a framework-sized service locator. If a test requires a complicated mock graph for a pure component, re-check the boundary.

## Debugging lab: circular dependency

If `CatalogPage` imports a shared `formatMoney`, while `shared/lib/formatMoney` imports `CatalogPage` to obtain its `Money` type, the dependency direction is reversed. Move the pure shared type to the narrow domain contract, or let the formatter accept a primitive amount/currency shape. Avoid runtime imports for type-only values with `import type`.

## Exercises

1. Draw the current module dependency graph. Add `account` and `admin-products` without allowing shared UI to depend on them.
2. Refactor the product header action from a boolean-heavy component into composition.
3. Identify a type that appears in two features; decide whether it is truly shared or duplicated intentionally.
4. Run TypeScript/ESLint and resolve a circular import or unused public export.
5. Write an architecture decision note for a feature you are postponing and why.

## Production tips, summary, and mental model

- Structure around business workflows and change frequency, not speculative layers.
- Keep route components thin enough to understand; keep complex behavior in feature-owned hooks/functions.
- Shared abstractions must have stable names and semantics; otherwise keep them local.
- Use an explicit dependency direction and review it when a feature imports another feature's internals.
- Do not let UI components make authorization or price calculations authoritative.

```text
feature owns its workflow
shared code points inward only to stable primitives
API boundary owns transport + parsing
server owns durable truth and permissions
```

**Summary:** Architecture is a set of ownership and dependency decisions. Feature modules keep related behavior close; composition creates flexible UI APIs; stable shared code is extracted when justified.

## Official documentation

- [Thinking in React](https://react.dev/learn/thinking-in-react)
- [Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)
- [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
- [React TypeScript: Passing Props](https://react.dev/learn/typescript)
- [TypeScript Modules](https://www.typescriptlang.org/docs/handbook/2/modules.html)
- [TypeScript `import type`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html)

## Readiness criteria

You can explain ownership for page, feature, shared UI, API, and server logic; draw a directed dependency graph; decide when not to abstract; and add one feature without making every module import the application root.
