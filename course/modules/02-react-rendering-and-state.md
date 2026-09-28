# Module 2 — React Rendering, Components, Props, Events, and State

> **Build goal:** turn Module 1's static catalogue into an interactive catalogue with search, category filtering, sorting, and a clear no-results state. Keep the same product fixtures; do not install a global store, router, or data library yet.

## Why this module matters

React is most useful when we stop describing a page as a pile of DOM commands and instead describe the interface as a function of current inputs. React can then compare the new description with the previous one and commit the necessary changes. To reason about bugs, distinguish **rendering** (calculate what should appear) from **committing** (apply changes to the host environment, the browser DOM here).

```text
props + state + context
          ↓
  component render (pure calculation)
          ↓
 React reconciles the result
          ↓
 DOM commit → browser paint
```

A component render may happen even when no visible pixels change. A render is not a promise that one DOM node will be created, nor is it a place to mutate global data or perform network requests.

## 1. Components and data flow

A React function component receives inputs, usually props, and returns JSX. Props flow from a parent to a child. A child requests a change by calling a callback provided by its owner; it does not reach into a parent and mutate its data.

```tsx
interface SearchBoxProps {
  value: string;
  onValueChange: (next: string) => void;
}

function SearchBox({ value, onValueChange }: SearchBoxProps) {
  return (
    <label>
      Search products
      <input
        type="search"
        value={value}
        onChange={(event) => onValueChange(event.currentTarget.value)}
      />
    </label>
  );
}
```

`SearchBox` is controlled: the parent owns the value, and the input reflects it. A small reusable input may also be uncontrolled. Neither is morally superior. Choose based on who must observe or coordinate the value. If a form owns all fields and only reads them on submission, uncontrolled inputs can avoid redundant state. If several controls must stay in sync, controlled values can make ownership clearer.

JSX is syntax for describing elements. `{expression}` evaluates JavaScript; it does not execute arbitrary HTML. Use real `<button>` elements for actions and `<a>`/router links for navigation.

## 2. State is a render snapshot

`useState` gives a component a value for the current render and a setter that asks React to render again with a future value.

```tsx
const [search, setSearch] = useState("");
```

Calling `setSearch("lamp")` does not mutate the `search` variable inside the currently running handler. That variable is a snapshot from the render that created the handler. React queues updates and may batch several together.

When the next value depends on the previous value, express the relationship as an updater:

```tsx
setPage((currentPage) => currentPage + 1);
```

This is safer than `setPage(page + 1)` when multiple updates may be queued. For independent replacement values, a direct setter is clearer.

**State-design rule:** do not store a value React can calculate from the current props/state. Store the smallest independent facts; derive the rest during render.

## 3. Derive a view without duplicating state

The parent owns the query and category selection. Visible products are derived from those values and the source data.

```tsx
const [search, setSearch] = useState("");
const [category, setCategory] = useState<ProductCategory | "all">("all");

const needle = search.trim().toLocaleLowerCase();
const filteredProducts = demoProducts.filter(
  (product) =>
    (category === "all" || product.category === category) &&
    `${product.name} ${product.description}`.toLocaleLowerCase().includes(needle),
);
const visibleProducts = [...filteredProducts].sort(
  (a, b) => a.price.amountMinor - b.price.amountMinor,
);
```

`sort` mutates its receiver, so the example copies before sorting. `toSorted` is a concise non-mutating alternative in ES2023-capable targets, but Module 1 deliberately targets ES2022, so this course uses the explicit copy. Filtering and sorting are deterministic calculations, not external synchronization, so an Effect plus a second `visibleProducts` state variable would add a redundant render and synchronization risk.

A simplified page composition:

```tsx
function parseCategory(value: string): ProductCategory | "all" {
  if (value === "audio" || value === "home" || value === "office") return value;
  return "all";
}

function CatalogPage() {
  const [search, setSearch] = useState("");
  const [category, setCategory] = useState<ProductCategory | "all">("all");
  const products = selectCatalogProducts(demoProducts, search, category);

  return (
    <main>
      <h1>Catalogue</h1>
      <SearchBox value={search} onValueChange={setSearch} />
      <label>
        Category
        <select
          value={category}
          onChange={(event) => setCategory(parseCategory(event.currentTarget.value))}
        >
          <option value="all">All categories</option>
          <option value="audio">Audio</option>
          <option value="home">Home</option>
          <option value="office">Office</option>
        </select>
      </label>
      <p aria-live="polite">{products.length} products found</p>
      {products.length === 0 ? (
        <section aria-labelledby="empty-title">
          <h2 id="empty-title">No products match</h2>
          <p>Try another word or clear the category filter.</p>
        </section>
      ) : (
        <ProductGrid products={products} />
      )}
    </main>
  );
}
```

`parseCategory` narrows the string against an explicit allowlist. TypeScript alone cannot prove that a DOM string is a valid category; it comes from a runtime boundary. We will use Zod for larger schemas later.

## 4. Lists, identity, and keys

A key tells React which logical item a rendered child represents across inserts, deletes, and reorderings.

```tsx
<ul>
  {products.map((product) => (
    <li key={product.id}>
      <ProductCard product={product} />
    </li>
  ))}
</ul>
```

Use a stable domain identifier. An array index is only valid if the list can never be reordered, inserted into, deleted from, or filtered—and that assumption is often broken later. `useId` is for connecting accessibility relationships such as `aria-describedby`; it is not a list-key generator.

## 5. Events and state ownership

Use event handlers for work caused by a user's action. A click is not an Effect. A derived price is not an Effect. An Effect is for synchronizing with something outside React; Module 3 develops that boundary.

```tsx
<button type="button" onClick={() => setCategory("audio")}>
  Audio
</button>
```

If a child needs to request a change, pass a callback. If unrelated parts of the page need the same state, lift it to their closest common owner. Do not lift every value to the root “just in case.”

### Debugging lab: a stale state update

```tsx
function addTwo() {
  setCount(count + 1);
  setCount(count + 1);
}
```

Both expressions use the same render snapshot, so this often increments only once. Use `setCount((current) => current + 1)` twice when two increments are intended. If the bug is actually “we should increment once,” one updater is enough. Always fix the intended domain action, not merely the observed number.

## Build checklist

- Search is case-insensitive and ignores leading/trailing whitespace.
- Category and search can be changed independently.
- Sorting does not mutate `demoProducts`.
- Empty results are distinct from loading or failure.
- Search and filter controls have labels, keyboard focus is visible, and status is not communicated by color alone.
- Product cards still use stable keys and accept their data through props.

## Exercises

1. Add a sort control for price ascending/descending. Keep the sort choice as the smallest state needed; do not mutate the fixture array.
2. Add a “Clear filters” action. Ensure it is a button, not a fake link.
3. Write a pure `selectCatalogProducts` test for whitespace, case, category, and sorting.
4. Add a test where no results are found and the empty-state heading appears.
5. **Debug:** store `visibleProducts` in state and synchronize it with an Effect. Explain the extra render and stale-result risk, then remove the Effect.

## Production tips and common mistakes

- Keep values local until a real consumer needs them elsewhere.
- Keep computed values in render; memoize only after profiling shows the calculation is costly.
- Use stable keys from data, not list positions or `useId`.
- Avoid deeply nested state; normalized data and explicit identifiers make updates easier.
- Avoid passing unrelated props through many layers; first check whether composition or a focused context is more appropriate.
- Never mutate a prop or module-level fixture from an event handler.

## Summary and mental model

**Summary:** Components describe UI; props communicate inputs; state records independent facts; events request updates; React renders a new snapshot; keys preserve item identity. Derived UI should be calculated rather than mirrored into state.

**Mental model:**

```text
user event → setter/callback → new render snapshot
source facts + props → pure derived view → keyed JSX → DOM commit
```

## Official documentation

- [Your First Component](https://react.dev/learn/your-first-component)
- [Thinking in React](https://react.dev/learn/thinking-in-react)
- [Responding to Events](https://react.dev/learn/responding-to-events)
- [State as a Snapshot](https://react.dev/learn/state-as-a-snapshot)
- [Queueing a Series of State Updates](https://react.dev/learn/queueing-a-series-of-state-updates)
- [Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)
- [Rendering Lists and Keys](https://react.dev/learn/rendering-lists)
- [React `useState`](https://react.dev/reference/react/useState)
- [React `useId`](https://react.dev/reference/react/useId)
- [React with TypeScript](https://react.dev/learn/typescript)

## Readiness criteria

Move on when you can explain why an event's `count` may be stale after calling a setter, identify the correct owner of search/filter state, derive a view without an Effect, explain the purpose of a key, and build the interactive catalogue from the Module 1 fixture without copying the reference blindly.
