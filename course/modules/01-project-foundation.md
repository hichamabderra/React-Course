# Module 1 — Project Foundation, JavaScript Mental Models, and the TypeScript Bridge

> **Course baseline:** Node.js 24.21.0 LTS, React/React DOM 19.3.0, Vite 8.3.1, TypeScript 6.0.3, ESLint 10.11.0. See [Verified Stack](../verified-stack.md) for why TypeScript 6 is the starter pin even though TypeScript 7 is the current stable compiler.

## What you will build

A small, static but production-minded storefront catalogue for **Northstar Commerce**. It will use typed product data, semantic HTML, stable list keys, responsive CSS, and a repeatable toolchain. It has no fake API, unnecessary global store, or decorative React state yet. Interactivity and remote data arrive in later modules, after we understand who should own them.

**Estimated effort:** one focused session for setup and the reference build, then additional time for the exercises. Do not measure progress by how quickly you can paste the code. Measure it by whether you can explain and change it without breaking its boundaries.

## Learning goals

By the end of this module, you should be able to:

- Explain the difference between JavaScript, React, React DOM, TypeScript, and Vite.
- Create a small Vite/React/TypeScript project with exact dependencies, a committed lockfile, Git, linting, formatting, a typecheck, and a production build.
- Use JavaScript modules, destructuring, spread/rest, closures, array transformations, promises, `async`/`await`, `fetch`, HTTP status checks, and `AbortSignal` accurately.
- Explain why mutating an existing array/object is risky in UI code and use immutable transformations instead.
- Describe what TypeScript can catch and what it cannot validate at runtime.
- Build a semantic product list with typed props and stable keys.
- Run the same project checks locally that a teammate or CI job can run.

---

## 1. First principles: what are these tools?

| Technology | What it is | What it is not |
|---|---|---|
| **JavaScript** | The programming language that runs in the browser (and in Node.js for tools/server code). | Not React; the language has no built-in component tree or JSX semantics. |
| **React** | A declarative UI library. Components describe the UI for current inputs/state; React schedules rendering and applies changes through a renderer. | Not a router, API client, server, CSS framework, or database. |
| **React DOM** | React's browser DOM renderer. `createRoot` connects a React tree to a browser DOM container. | Not the same package as the core `react` package. Keep `react` and `react-dom` versions aligned. |
| **TypeScript** | JavaScript plus a static type system and compiler tooling. Types help humans and tools find mistakes before execution. | Not a runtime validator. An `interface` is erased; it does not inspect JSON received over HTTP. |
| **Vite** | A development server and build tool. It serves modules/HMR during development and bundles/transforms the app for production. | Not React itself and not a backend. Vite's dev server is not the server your customers use. |

### Mental model: the browser toolchain

```text
.tsx source + CSS
      │
      ├── TypeScript checker (tsc): diagnoses type errors; does not normally emit our app
      ├── Vite/Oxc + React plugin: transforms TS/JSX and serves HMR during development
      └── Vite production build: bundles/minifies static browser assets
                        │
                        ▼
              React element/component tree
                        │
             React render → reconcile → commit
                        │
                        ▼
                  Browser DOM + paint
```

A user action enters JavaScript through a browser event. React state updates schedule rendering; React computes what the UI should be, then commits only the necessary DOM updates. A component render is not a command to “replace the page.” We will build this model in depth in Module 2.

### What React is versus what TypeScript is

| Question | React's answer | TypeScript's answer |
|---|---|---|
| “What should this screen look like for this input?” | Component function returns JSX. | It can check that the input/output shapes are used consistently. |
| “What changes the visible interface?” | State/props/context updates cause React to render again. | Types do not render anything and do not exist at runtime. |
| “Can this API response really be a `Product[]`?” | No. React renders values passed to it. | No. A type annotation or `as Product[]` is erased. Validate `unknown` data at runtime (Zod in a later module). |
| “Who inserts/updates browser DOM nodes?” | React DOM. | Neither TypeScript nor Vite. |

A productive habit: when an error mentions a value's type, ask whether the fix belongs to **React's data flow** or **TypeScript's compile-time model**. They frequently interact, but they solve different problems.

---

## 2. JavaScript foundations React engineers actually use

React code is JavaScript. TypeScript does not change JavaScript's runtime behavior; it gives the editor/checker more information. These are the language patterns we will use in the project.

### Modules, imports, and destructuring

Keep feature code in modules with explicit imports. Prefer named exports for the parts of a feature other modules are expected to use. Import types with `import type` so the runtime/build pipeline can erase type-only imports cleanly.

```ts
import { demoProducts } from "./demo-products";
import type { Product } from "./product";

function ProductSummary({ name, price }: { name: string; price: Product["price"] }) {
  return <p>{name}: {price.amountMinor}</p>;
}
```

Destructuring is a readable way to take values out of a well-defined object. Do not destructure a value before checking whether it can be `null` or `undefined`.

### Spread/rest and immutability

Spread creates a **shallow** copy. That is enough for a shallow update, but nested objects must be copied at every level that changes.

```ts
interface CartLine {
  productId: string;
  quantity: number;
}

function changeQuantity(lines: readonly CartLine[], productId: string, quantity: number) {
  return lines.map((line) =>
    line.productId === productId ? { ...line, quantity } : line,
  );
}
```

The input is `readonly` to communicate “this function reads the caller's data; it does not own permission to mutate it.” The result is a new array. Unchanged line objects can be reused safely because they are not being modified.

**Bad:**

```ts
function changeQuantityBad(lines: CartLine[], productId: string, quantity: number) {
  const line = lines.find((item) => item.productId === productId);
  if (line) line.quantity = quantity; // mutates an object held by the caller
  return lines; // returns the same array identity
}
```

This is not only a “React rule.” It is a data-ownership bug: any other code holding `lines` sees the mutation without being told that a new value was produced. React relies heavily on identity to detect changes efficiently, so state mutations also make UI updates harder to reason about.

Rest syntax gathers remaining values. It is useful for explicit patch APIs, but can accidentally copy fields that should not be writable. Later we will use allowlisted schemas for updates.

### `map`, `filter`, `reduce`, and sorting without mutation

```ts
const names = demoProducts.map((product) => product.name);
const available = demoProducts.filter((product) => product.stock !== "sold-out");
const inventoryValueMinor = demoProducts.reduce(
  (total, product) => total + product.price.amountMinor,
  0,
);

// Array.prototype.sort mutates its receiver. Copy first when the source is shared.
const byPrice = [...demoProducts].sort(
  (left, right) => left.price.amountMinor - right.price.amountMinor,
);
```

`map` is for one-to-one transformation, `filter` chooses a subset, and `reduce` folds values into an accumulator. Do not use `reduce` for every loop just because it is available; a `for...of` loop is often clearer when there are several branches or side effects.

### Closures and higher-order functions

A closure is a function plus access to the lexical variables from the scope where it was created. A higher-order function accepts or returns another function.

```js
function makeDiscount(rate) {
  return function discountCents(amountMinor) {
    return Math.round(amountMinor * (1 - rate));
  };
}

const memberPrice = makeDiscount(0.1);
memberPrice(12_900); // 11610
```

This is ordinary JavaScript. It is also the foundation of callbacks, event handlers, and custom hooks. Closures retain the values from the render in which a React handler was created. That fact is useful; it is also the source of stale-closure bugs if asynchronous work outlives the assumptions made by that render. Module 3 will debug that case.

### Promises, `async`/`await`, and the event loop

A Promise represents the eventual result (or failure) of asynchronous work. `await` makes Promise code read sequentially; it does not block the browser's JavaScript thread while waiting for network I/O.

```js
async function getRecommendations() {
  const [popular, featured] = await Promise.all([
    fetchPopularProducts(),
    fetchFeaturedProducts(),
  ]);

  return [...popular, ...featured];
}
```

`Promise.all` is appropriate when all results are required. If these calls are independent and a page can show partial data, `Promise.allSettled` or separate queries may produce a better user experience. A rejected Promise in `Promise.all` rejects the aggregate result.

A useful simplified timeline:

```text
browser event task
  └─ handler starts fetch(), receives Promise, returns
network response arrives
  └─ Promise continuation runs as a microtask
      └─ application records result / React schedules an update
browser gets an opportunity to render and paint
```

The event loop is not “one thread per `async` function.” JavaScript execution still runs to completion for each call stack; the browser coordinates events, microtasks, rendering, and network activity. Do not put expensive synchronous work into a keypress handler and expect `async` to make the CPU work non-blocking.

### `fetch`, status codes, errors, and cancellation

`fetch()` rejects for network-level failures and aborts. It normally **resolves** for HTTP `404`, `401`, `403`, and `500` responses. The caller must check `response.ok` or `response.status`.

```ts
async function readJson(url: string, signal: AbortSignal): Promise<unknown> {
  const response = await fetch(url, {
    signal,
    credentials: "include",
    headers: { Accept: "application/json" },
  });

  if (!response.ok) {
    throw new Error(`Request failed with HTTP ${response.status}`);
  }

  // JSON parsing does not establish that the value matches a TypeScript type.
  const payload: unknown = await response.json();
  return payload;
}
```

`AbortController` gives the caller a cancellation signal:

```ts
const controller = new AbortController();

const request = fetch("/api/products", { signal: controller.signal });

// If the user navigates away or starts a newer search:
controller.abort();
```

Cancellation matters for typeahead and navigation: an old response should not overwrite a newer request's result. Aborting stops the browser client from continuing to wait/consume the response; it does not guarantee that a server-side operation that already began was undone. Mutating requests still need correct idempotency and server semantics.

The client code uses the **relative** path `/api/products`. It does not call `http://localhost:4000` from browser code. In development Vite can proxy `/api` to a local backend; in production a reverse proxy can route `/api` to the deployed service.

### Optional chaining and nullish coalescing

```ts
const city = user.profile?.address?.city ?? "Not provided";
```

`?.` safely stops at `null`/`undefined`. `??` uses the fallback only for `null` or `undefined`; unlike `||`, it preserves valid values such as `0`, `false`, and `""`. Do not use either operator to conceal an invalid state that should be handled explicitly.

---

## 3. TypeScript: progressively, not ceremonially

TypeScript adds static types to JavaScript expressions. A type helps the compiler and editor reason about source code; it is not a runtime object validator and does not change React's rendering model.

### A small domain type

```ts
export type ProductCategory = "audio" | "home" | "office";
export type StockStatus = "in-stock" | "low-stock" | "sold-out";

export interface Money {
  amountMinor: number;
  currency: "USD";
}

export interface Product {
  id: string;
  name: string;
  description: string;
  category: ProductCategory;
  price: Money;
  stock: StockStatus;
}
```

- Use `interface` or `type` for object shapes; do not turn the choice into a religious rule. A `type` is convenient for unions/intersections; an `interface` is often pleasant for an extensible object contract.
- The string union is a **discriminated set of allowed values**, not a runtime enum object.
- `Money` stores integer minor units for the capstone's USD-only examples. A real multi-currency system must define the minor-unit exponent and arithmetic/rounding rules per currency; do not store money in binary floating point as the authoritative value.

### Narrowing and exhaustive cases

```ts
function stockMessage(stock: StockStatus): string {
  switch (stock) {
    case "in-stock":
      return "Available now";
    case "low-stock":
      return "Limited availability";
    case "sold-out":
      return "Currently unavailable";
  }
}
```

A union narrows as control flow eliminates possibilities. TypeScript can verify that known union branches are handled. When an API sends an arbitrary string, the runtime still needs to prove that it is one of these values before the application trusts it.

### Compile-time type versus runtime value

```ts
type User = { id: string; email: string };

const user = JSON.parse("{\"id\": 7, \"email\": null}") as User;
// Compiles after the assertion. At runtime: id is still a number and email is still null.
```

`as User` tells TypeScript “believe me.” It does **not** convert, validate, or sanitize data. Prefer to treat remote input as `unknown`, parse it at the boundary, and expose a trusted result only after validation. Zod and the API contract are covered later.

### Generics describe reusable relationships

```ts
function firstOrUndefined<T>(items: readonly T[]): T | undefined {
  return items[0];
}

const firstProduct = firstOrUndefined(demoProducts); // Product | undefined
const firstName = firstOrUndefined(["Ava", "Kai"]); // string | undefined
```

`T` says the output has a relationship to the input type. It does not mean “make every function generic.” Add a generic when callers genuinely vary the type while preserving a useful relationship.

### TypeScript 7 is a real current change

TypeScript 7.0 is the stable Go-native compiler release and reports major compiler/tooling performance gains. That is important current knowledge. The starter still uses TS6.0.3 because the selected current `typescript-eslint` line officially supports `<6.1.0` and TS7.0 lacks the stable programmatic API that the TypeScript-aware lint tool depends on. We will keep the React concepts independent of compiler implementation, compare the TS7 migration requirements, and change the project only when the project tooling supports it.

---

## 4. Professional setup and repeatable checks

### Check the runtime first

Install/select Node 24.21.0 LTS using the version manager your team supports, then verify:

```bash
node --version
npm --version
git --version
```

Expected Node result for this course pin: `v24.21.0`. Keep the runtime selection in `.nvmrc` (or the equivalent file used by your team). Do not rely on a globally installed TypeScript compiler: the project-local compiler and lockfile should determine the result.

### Scaffold, then pin the actual libraries

The scaffold command creates a React + TypeScript Vite project; it is a convenience, not a version policy. We normalize the key dependencies afterward to the verified versions.

```bash
npm create vite@latest northstar-commerce -- --template react-ts
cd northstar-commerce

npm install --save-exact react@19.3.0 react-dom@19.3.0
npm install --save-dev --save-exact \
  vite@8.3.1 @vitejs/plugin-react@6.1.1 \
  typescript@6.0.3 @types/react@19.3.0 @types/react-dom@19.3.0 @types/node@24.19.0 \
  eslint@10.11.0 @eslint/js@10.0.1 typescript-eslint@8.70.1 \
  eslint-plugin-react-hooks@7.1.1 eslint-plugin-react-refresh@0.5.7 globals@17.12.0 \
  prettier@3.9.9
```

If you use a different package manager, use its exact-version syntax and commit its one lockfile. Do not commit multiple lockfiles for the same app.

### File: `.nvmrc`

```text
24.21.0
```

### File: `.npmrc`

```ini
save-exact=true
```

Exact direct dependency versions plus a lockfile make the initial course project repeatable. They do not replace dependency review or patch updates; a team should update intentionally and run its checks.

### File: `package.json`

The important script contract is that a developer can run the same checks locally and in CI. The full initial manifest is:

```json
{
  "name": "northstar-commerce",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "engines": {
    "node": ">=24.21.0 <25"
  },
  "scripts": {
    "dev": "vite --host 0.0.0.0",
    "build": "tsc -b && vite build",
    "typecheck": "tsc -b --pretty false",
    "lint": "eslint .",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "preview": "vite preview --host 0.0.0.0"
  },
  "dependencies": {
    "react": "19.3.0",
    "react-dom": "19.3.0"
  },
  "devDependencies": {
    "@eslint/js": "10.0.1",
    "@types/node": "24.19.0",
    "@types/react": "19.3.0",
    "@types/react-dom": "19.3.0",
    "@vitejs/plugin-react": "6.1.1",
    "eslint": "10.11.0",
    "eslint-plugin-react-hooks": "7.1.1",
    "eslint-plugin-react-refresh": "0.5.7",
    "globals": "17.12.0",
    "prettier": "3.9.9",
    "typescript": "6.0.3",
    "typescript-eslint": "8.70.1",
    "vite": "8.3.1"
  }
}
```

The starter intentionally does not install Query, Zustand, shadcn, Motion, or test libraries yet. Add a dependency when the module introduces the problem it solves. After editing the manifest, run `npm install` to produce/update `package-lock.json`, then commit both files.

### File: `vite.config.ts`

```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
});
```

Vite will later proxy the relative `/api` path in development. The browser will still request `/api/...`; the proxy target is server-side development configuration. Do not “fix” a proxy by hard-coding `localhost` in frontend source. If a dev server has a host allowlist, permit the intended development/preview hostname narrowly; do not set a blanket unsafe host bypass.

### File: `src/vite-env.d.ts`

The React + TypeScript scaffold includes Vite's client declarations. Keep the reference so `import.meta.env` and Vite-provided asset types are checked in browser code:

```ts
/// <reference types="vite/client" />
```

Do not put secrets in `import.meta.env`; client-exposed values are public. Module 19 uses `import.meta.env.DEV` only to keep profiling logs in development.

### TypeScript settings worth understanding

The Vite template uses separate project configuration for app code and tool configuration. Preserve the template's project references and keep application compiler options strict. The key app-code choices are:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,
    "noEmit": true
  },
  "include": ["src"]
}
```

- `strict` makes TypeScript ask us to handle uncertainty rather than silently widening everything.
- `noUncheckedIndexedAccess` reminds us that indexing an array/object may produce no value.
- `moduleResolution: "bundler"` models the package/export resolution expected by Vite.
- `isolatedModules` helps ensure each file can be transformed independently.
- `verbatimModuleSyntax` makes runtime imports versus `import type` explicit.
- `noEmit` means Vite builds the browser assets; TypeScript checks types without emitting a second competing set of JavaScript files.

**Critical tooling distinction:** Vite 8 transforms TypeScript syntax but does **not** type-check it. `npm run build` therefore runs TypeScript's project build first and only then Vite's production build. Keep a separate `npm run typecheck` command for fast feedback and clear CI failure reporting. See [Vite's TypeScript documentation](https://vite.dev/guide/features#typescript).

### File: `eslint.config.js`

ESLint 10 uses flat config. The `typescript-eslint` package provides the parser and recommended TypeScript rules. React's maintained Hooks plugin also surfaces Compiler-related diagnostics, even before we enable compilation.

```js
import js from "@eslint/js";
import globals from "globals";
import { defineConfig } from "eslint/config";
import reactHooks from "eslint-plugin-react-hooks";
import reactRefresh from "eslint-plugin-react-refresh";
import tseslint from "typescript-eslint";

export default defineConfig([
  { ignores: ["dist", "coverage", ".vitest"] },
  {
    files: ["src/**/*.{ts,tsx}"],
    extends: [
      js.configs.recommended,
      tseslint.configs.recommended,
      reactHooks.configs.flat.recommended,
    ],
    languageOptions: {
      globals: globals.browser,
    },
    plugins: {
      "react-refresh": reactRefresh,
    },
    rules: {
      "react-refresh/only-export-components": [
        "warn",
        { allowConstantExport: true },
      ],
    },
  },
  {
    files: ["vite.config.ts"],
    extends: [js.configs.recommended, tseslint.configs.recommended],
    languageOptions: {
      globals: globals.node,
    },
  },
]);
```

This is a small starter configuration, not a claim that every team needs identical rules. When test files arrive, add their globals/setup deliberately. Avoid disabling `exhaustive-deps` because it finds inconvenient dependency changes; first determine whether the effect belongs in an event, a render-time calculation, or a genuine external synchronization.

### File: `.gitignore`

Keep generated output and local secrets out of Git. Preserve `package-lock.json` in Git.

```gitignore
node_modules/
dist/
coverage/
.vitest/
storybook-static/
.env
.env.*
!.env.example
*.local
```

Never place passwords, signing keys, private API keys, or long-lived credentials into `VITE_*` variables or the browser bundle. Vite documents that `VITE_*` values are exposed to client code at build time. Use server-side configuration for secrets.

### Run the quality gates

```bash
npm run typecheck
npm run lint
npm run format:check
npm run build
npm run dev
```

Then use the browser's responsive device toolbar, Console, and Network panel. Check a narrow viewport, keyboard focus visibility, page title, and that no request/asset is failing. `npm run preview` serves the built output; it is not a production backend.

---

## 5. Workshop: create the first catalogue

### Build this before looking at the reference implementation

Create a page with:

- A clear page title, site header, skip link, main content, labelled catalogue section, and footer.
- Three sample products rendered from data, not three copy-pasted cards.
- Product name, category, description, USD price in minor units, and a stock label.
- A responsive grid: one column on a narrow phone, two at a small tablet width, and three on a desktop.
- Keyboard-visible focus styles and sufficient text/background contrast.
- No product “Add to cart” button yet. There is no cart behavior yet, so showing a dead button would be misleading.

**Constraints:** no array index as a key, no direct DOM manipulation, no global store, no `useEffect`, no `any`, and no network call. Keep the data and product card separate from the page composition.

Think through the answer before opening the reference below:

1. Which component owns the list?
2. Which component knows how to display one product?
3. Which values are domain data, and which are display formatting?
4. How does the UI know which product a list row represents after sorting?

### Reference implementation

The following is a small, complete baseline—not the only valid folder structure.

#### File: `index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta
      name="description"
      content="Thoughtful everyday goods from Northstar Commerce."
    />
    <title>Northstar Commerce — curated everyday goods</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

#### File: `src/main.tsx`

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { App } from "./app/App";
import "./app/app.css";

const rootElement = document.getElementById("root");

if (!rootElement) {
  throw new Error('Expected the HTML document to contain an element with id="root".');
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

`createRoot` is the React DOM entry point for a client-rendered root. `StrictMode` is a development-time aid that can expose impure rendering and unsafe setup/cleanup. In development, React may intentionally call render logic more than once to reveal bugs; do not “fix” that by turning Strict Mode off. Production rendering is not double-run for this check.

#### File: `src/features/catalog/product.ts`

```ts
export type ProductCategory = "audio" | "home" | "office";
export type StockStatus = "in-stock" | "low-stock" | "sold-out";

export interface Money {
  amountMinor: number;
  currency: "USD";
}

export interface Product {
  id: string;
  name: string;
  description: string;
  category: ProductCategory;
  price: Money;
  stock: StockStatus;
}
```

The `Product` type is a compile-time contract for our local fixture and component props. In later modules, the HTTP response will start as `unknown`; we will not claim it is `Product` until a runtime schema accepts it.

#### File: `src/features/catalog/demo-products.ts`

```ts
import type { Product } from "./product";

export const demoProducts = [
  {
    id: "prod-1001",
    name: "Orbit Studio Headphones",
    description: "Clear, comfortable sound for focused work and long journeys.",
    category: "audio",
    price: { amountMinor: 12900, currency: "USD" },
    stock: "in-stock",
  },
  {
    id: "prod-1002",
    name: "Solstice Desk Lamp",
    description: "Warm, adjustable light with a small footprint.",
    category: "office",
    price: { amountMinor: 8400, currency: "USD" },
    stock: "low-stock",
  },
  {
    id: "prod-1003",
    name: "Harbor Ceramic Set",
    description: "A durable stoneware set made for everyday meals.",
    category: "home",
    price: { amountMinor: 5600, currency: "USD" },
    stock: "sold-out",
  },
] satisfies readonly Product[];
```

`satisfies` checks that each fixture matches the `Product` contract while retaining useful literal information. This is a good fit for static local data. It does not validate remote JSON at runtime.

#### File: `src/features/catalog/ProductCard.tsx`

```tsx
import type { Product, ProductCategory, StockStatus } from "./product";

interface ProductCardProps {
  product: Product;
}

const currencyFormatter = new Intl.NumberFormat("en-US", {
  style: "currency",
  currency: "USD",
});

const artClassByCategory: Record<ProductCategory, string> = {
  audio: "product-art--audio",
  home: "product-art--home",
  office: "product-art--office",
};

const stockLabel: Record<StockStatus, string> = {
  "in-stock": "In stock",
  "low-stock": "Low stock",
  "sold-out": "Sold out",
};

function formatPrice(product: Product): string {
  // This example is USD-only, so its minor unit is one cent.
  return currencyFormatter.format(product.price.amountMinor / 100);
}

export function ProductCard({ product }: ProductCardProps) {
  return (
    <article className="product-card">
      <div
        className={`product-art ${artClassByCategory[product.category]}`}
        aria-hidden="true"
      >
        <span className="product-art__label">Northstar / {product.category}</span>
      </div>

      <div className="product-card__body">
        <p className="eyebrow">{product.category}</p>
        <h3 className="product-card__title">{product.name}</h3>
        <p className="product-card__description">{product.description}</p>

        <div className="product-card__meta">
          <p className="product-card__price">{formatPrice(product)}</p>
          <p className={`stock-status stock-status--${product.stock}`}>
            <span className="stock-status__dot" aria-hidden="true" />
            {stockLabel[product.stock]}
          </p>
        </div>
      </div>
    </article>
  );
}
```

The decorative illustration is hidden from assistive technology because the product name and category already provide the relevant information. The visible stock words are not represented by color alone. `Intl.NumberFormat` is built into the platform; for a real multi-currency app, the server contract must state currency and minor-unit rules explicitly.

#### File: `src/app/App.tsx`

```tsx
import { demoProducts } from "../features/catalog/demo-products";
import { ProductCard } from "../features/catalog/ProductCard";

export function App() {
  return (
    <>
      <a className="skip-link" href="#main-content">
        Skip to main content
      </a>

      <div className="app-shell">
        <header className="site-header">
          <a className="brand" href="/" aria-label="Northstar Commerce home">
            <span className="brand__mark" aria-hidden="true">
              N
            </span>
            <span>
              Northstar <strong>Commerce</strong>
            </span>
          </a>

          <nav className="primary-nav" aria-label="Main navigation">
            <a href="#catalog">Catalogue</a>
            <a href="#about">About</a>
          </nav>

          <span className="header-note">Storefront preview</span>
        </header>

        <main id="main-content" className="main-content" tabIndex={-1}>
          <section className="hero" aria-labelledby="hero-title">
            <p className="eyebrow">Thoughtful goods. Straightforward shopping.</p>
            <h1 id="hero-title">Useful things, chosen with care.</h1>
            <p className="hero__description">
              A growing catalogue for everyday work, home, and travel.
            </p>
          </section>

          <section id="catalog" className="catalog" aria-labelledby="catalog-title">
            <div className="section-heading">
              <div>
                <p className="eyebrow">A small first collection</p>
                <h2 id="catalog-title">Popular right now</h2>
              </div>
              <p className="catalog-count">
                {demoProducts.length} products
              </p>
            </div>

            <ul className="product-grid">
              {demoProducts.map((product) => (
                <li key={product.id}>
                  <ProductCard product={product} />
                </li>
              ))}
            </ul>
          </section>

          <section id="about" className="about" aria-labelledby="about-title">
            <h2 id="about-title">Designed for shoppers and teams</h2>
            <p>
              This catalogue is the first milestone. We will add routes, real API
              data, customer accounts, and admin workflows one feature at a time.
            </p>
          </section>
        </main>

        <footer className="site-footer">
          <p>Northstar Commerce · Course capstone</p>
        </footer>
      </div>
    </>
  );
}
```

The `App` composes a page; `ProductCard` knows how to render one product. The stable `product.id` key identifies each list item when React compares one rendered list with a later one. Array positions are not identities: sorting, inserting, or deleting can make index keys attach state/focus to the wrong row.

#### File: `src/app/app.css`

```css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #17212f;
  background: #f4f6f8;
  font-synthesis: none;
  text-rendering: optimizeLegibility;
  font-weight: 400;
  line-height: 1.5;

  --surface: #ffffff;
  --surface-muted: #e9eff2;
  --text-muted: #536273;
  --border: #d8e0e6;
  --brand: #176b64;
  --focus: #075fe4;
  --radius-card: 1.15rem;
}

* {
  box-sizing: border-box;
}

html {
  min-width: 320px;
  scroll-behavior: smooth;
}

body {
  min-width: 320px;
  margin: 0;
}

button,
a {
  font: inherit;
}

a {
  color: inherit;
}

a:focus-visible,
button:focus-visible,
main:focus-visible {
  outline: 3px solid var(--focus);
  outline-offset: 3px;
}

.skip-link {
  position: absolute;
  z-index: 10;
  inset-block-start: 0.75rem;
  inset-inline-start: 0.75rem;
  transform: translateY(-180%);
  border-radius: 0.5rem;
  padding: 0.7rem 1rem;
  color: #ffffff;
  background: #132c37;
}

.skip-link:focus {
  transform: translateY(0);
}

.app-shell {
  width: min(100% - 2rem, 1160px);
  margin-inline: auto;
}

.site-header {
  display: flex;
  min-height: 5.25rem;
  align-items: center;
  justify-content: space-between;
  gap: 1.5rem;
  border-bottom: 1px solid var(--border);
}

.brand {
  display: inline-flex;
  align-items: center;
  gap: 0.7rem;
  text-decoration: none;
  font-size: 1rem;
  letter-spacing: -0.02em;
}

.brand strong {
  font-weight: 750;
}

.brand__mark {
  display: grid;
  width: 2.15rem;
  aspect-ratio: 1;
  place-items: center;
  border-radius: 0.7rem;
  color: #ffffff;
  background: var(--brand);
  font-weight: 800;
}

.primary-nav {
  display: flex;
  gap: 1.25rem;
}

.primary-nav a {
  color: #344454;
  text-decoration-thickness: 0.08em;
  text-underline-offset: 0.22em;
}

.header-note,
.catalog-count {
  color: var(--text-muted);
  font-size: 0.875rem;
}

.main-content {
  min-height: 60vh;
}

.hero {
  padding-block: clamp(3.5rem, 8vw, 6.5rem) 3rem;
}

.eyebrow {
  margin: 0 0 0.65rem;
  color: var(--brand);
  font-size: 0.78rem;
  font-weight: 750;
  letter-spacing: 0.09em;
  text-transform: uppercase;
}

.hero h1 {
  max-width: 12ch;
  margin: 0;
  font-size: clamp(2.4rem, 7vw, 5.25rem);
  letter-spacing: -0.065em;
  line-height: 0.98;
}

.hero__description {
  max-width: 42rem;
  margin-block: 1.35rem 0;
  color: var(--text-muted);
  font-size: clamp(1rem, 2vw, 1.2rem);
}

.catalog {
  padding-block: 1.5rem 4.5rem;
}

.section-heading {
  display: flex;
  align-items: end;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 1.25rem;
}

.section-heading h2,
.about h2 {
  margin: 0;
  font-size: clamp(1.5rem, 3vw, 2rem);
  letter-spacing: -0.04em;
}

.catalog-count {
  margin: 0 0 0.15rem;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 1.15rem;
  margin: 0;
  padding: 0;
  list-style: none;
}

.product-card {
  height: 100%;
  overflow: hidden;
  border: 1px solid var(--border);
  border-radius: var(--radius-card);
  background: var(--surface);
}

.product-art {
  display: flex;
  min-height: 12rem;
  align-items: end;
  padding: 1rem;
  background: var(--surface-muted);
}

.product-art--audio {
  background: linear-gradient(135deg, #dce9e9, #a5c8c3);
}

.product-art--office {
  background: linear-gradient(135deg, #ece4d7, #d3bea0);
}

.product-art--home {
  background: linear-gradient(135deg, #eee0de, #d3aaa4);
}

.product-art__label {
  padding: 0.35rem 0.6rem;
  border: 1px solid rgb(23 33 47 / 18%);
  border-radius: 99px;
  background: rgb(255 255 255 / 78%);
  color: #17212f;
  font-size: 0.75rem;
  font-weight: 700;
}

.product-card__body {
  padding: 1.15rem;
}

.product-card__title {
  margin: 0;
  font-size: 1.15rem;
  letter-spacing: -0.025em;
}

.product-card__description {
  min-height: 3rem;
  margin-block: 0.55rem 1.1rem;
  color: var(--text-muted);
  font-size: 0.93rem;
}

.product-card__meta {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  border-top: 1px solid var(--border);
  padding-top: 0.9rem;
}

.product-card__price {
  margin: 0;
  font-weight: 750;
}

.stock-status {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  margin: 0;
  color: #344454;
  font-size: 0.82rem;
}

.stock-status__dot {
  width: 0.5rem;
  aspect-ratio: 1;
  border-radius: 50%;
  background: #177a46;
}

.stock-status--low-stock .stock-status__dot {
  background: #a65b00;
}

.stock-status--sold-out .stock-status__dot {
  background: #6a7480;
}

.about {
  border-top: 1px solid var(--border);
  padding-block: 2.25rem 3.5rem;
}

.about p {
  max-width: 48rem;
  color: var(--text-muted);
}

.site-footer {
  border-top: 1px solid var(--border);
  padding-block: 1.25rem;
  color: var(--text-muted);
  font-size: 0.85rem;
}

.site-footer p {
  margin: 0;
}

@media (max-width: 860px) {
  .product-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .header-note {
    display: none;
  }
}

@media (max-width: 600px) {
  .app-shell {
    width: min(100% - 1.25rem, 1160px);
  }

  .site-header {
    min-height: 4.5rem;
    flex-wrap: wrap;
    gap: 0.75rem;
    padding-block: 0.75rem;
  }

  .primary-nav {
    gap: 0.85rem;
    font-size: 0.9rem;
  }

  .product-grid {
    grid-template-columns: minmax(0, 1fr);
  }

  .product-art {
    min-height: 10rem;
  }

  .section-heading {
    align-items: start;
  }
}

@media (prefers-reduced-motion: reduce) {
  html {
    scroll-behavior: auto;
  }
}
```

The CSS is intentionally ordinary CSS for now. It gives us a baseline to compare against Tailwind v4 and shadcn later. It includes a visible focus indicator, a skip link, responsive layout, stock text as well as color, and a reduced-motion preference for smooth scrolling.

### Current project structure after the reference build

```text
northstar-commerce/
├── index.html
├── package.json
├── package-lock.json
├── vite.config.ts
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
├── eslint.config.js
├── .nvmrc
├── .npmrc
└── src/
    ├── vite-env.d.ts
    ├── main.tsx
    ├── app/
    │   ├── App.tsx
    │   └── app.css
    └── features/
        └── catalog/
            ├── demo-products.ts
            ├── ProductCard.tsx
            └── product.ts
```

### Current architecture

```text
src/main.tsx             browser entry point and React DOM root
       ↓
src/app/App.tsx          page-level composition and shared shell
       ↓
src/features/catalog/    catalogue fixture, product type, one-product view
```

These folders have a job, not an ideology. We have one small feature, so we do not yet create empty `services/`, `stores/`, `schemas/`, or `types/` folders. They will appear when the project has actual ownership boundaries that benefit from them.

### What we have built / what comes next

**Built:** a reproducible TypeScript Vite project; one static catalogue feature; a product view with explicit props; semantic page landmarks; responsive base CSS; lint, format, typecheck, and build commands.

**Not built yet:** routing, state, data fetching, forms, auth, admin actions, tests, or a design system. We have deliberately not invented their state/abstractions before we know their requirements.

**Next:** Module 2 makes the catalogue interactive with React state and events. We will first predict which component should own each state value, then implement it and verify the render behavior.

---

## 6. Debugging lab: understand the mistake, not just the patch

### Bug A — in-place sort changes shared source data

```ts
const sortedProducts = demoProducts.sort(
  (left, right) => left.price.amountMinor - right.price.amountMinor,
);
```

**Why it breaks:** `sort` mutates `demoProducts`, so rendering one view can change the source order used by another view. Mutating shared data makes render order and data ownership hard to predict.

**Fix:**

```ts
const sortedProducts = [...demoProducts].sort(
  (left, right) => left.price.amountMinor - right.price.amountMinor,
);
```

**Mental model:** a transformation should produce a new value when it changes the logical collection; do not silently rewrite an input owned elsewhere.

### Bug B — treating a TypeScript assertion as a parser

```ts
const products = (await response.json()) as Product[];
```

**Why it breaks:** the assertion is erased. A malformed response such as `{ "products": null }` remains malformed at runtime.

**Fix direction:** keep external values as `unknown`, validate at the API boundary (Zod in Module 13), and convert only validated results into domain types.

**Mental model:** static checking asks whether our source uses a claimed type consistently; runtime parsing asks whether a value from outside our program actually matches the contract.

### Bug C — assuming `fetch` rejects for `404`

```ts
const response = await fetch("/api/products/does-not-exist");
const product = await response.json(); // may be a 404 body, not a product
```

**Why it breaks:** HTTP error responses normally fulfill the Promise. The code tries to parse a response body as success without checking status.

**Fix direction:** check `response.ok`, normalize the error, and only parse success payloads. Keep `401` and `403` distinguishable; they mean different things.

### Preview bug D — effect for a derived value (the full lesson is Module 3)

```tsx
const [visibleProducts, setVisibleProducts] = useState<Product[]>([]);

useEffect(() => {
  setVisibleProducts(products.filter((product) => matches(product, search)));
}, [products, search]);
```

This stores a second copy of data React can calculate during render. It adds an extra render and creates a synchronization problem: `products`, `search`, and `visibleProducts` can temporarily disagree. For a small catalogue, use:

```tsx
const visibleProducts = products.filter((product) => matches(product, search));
```

There is no external system here. Effects are for synchronization with external systems; they are not a general “when anything changes” callback. The official React page [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect) covers this principle in detail.

---

## 7. Exercises

### Beginner exercise — price formatting and product state

1. Add a fourth product to `demo-products.ts`.
2. Add a `"limited"` category only if you can explain why it is a **domain category** rather than a display string. Otherwise use one of the existing categories.
3. Add a new stock value to `StockStatus` and `stockLabel`.
4. Verify TypeScript catches the missing label mapping before you run the page.

**Success check:** no `any`, no type assertion, and a clear stock phrase still appears without relying on color.

### Intermediate exercise — pure filtering and sorting

Create a pure function with this signature:

```ts
export function selectCatalogProducts(
  products: readonly Product[],
  searchText: string,
  category: ProductCategory | "all",
): Product[]
```

It should case-insensitively search the product name/description, optionally filter a category, sort by price ascending, and never mutate the input.

**Reference solution:**

```ts
export function selectCatalogProducts(
  products: readonly Product[],
  searchText: string,
  category: ProductCategory | "all",
): Product[] {
  const normalizedSearch = searchText.trim().toLocaleLowerCase();

  return products
    .filter((product) => {
      const matchesText =
        normalizedSearch.length === 0 ||
        `${product.name} ${product.description}`
          .toLocaleLowerCase()
          .includes(normalizedSearch);
      const matchesCategory = category === "all" || product.category === category;

      return matchesText && matchesCategory;
    })
    .slice()
    .sort((left, right) => left.price.amountMinor - right.price.amountMinor);
}
```

`.slice()` makes the sorting copy explicit. Current JavaScript also offers `toSorted`, but our project target is intentionally conservative and the array-copy pattern is widely understood. In Module 2, the component will own the search/category inputs and call this derived selector while rendering; we will not mirror the result into state with an Effect.

### Production-style exercise — API boundary design before code

Write the contract for a product-list request in a short note before implementing it:

- URL and query parameters for page, page size, category, search, and sort.
- Expected success status/body and at least one validation-error body.
- What `AbortSignal` cancels and what it cannot cancel.
- How the client handles 401, 403, 404, 409, 422, 429, 5xx, network failure, invalid JSON, and empty results.
- Which values are public environment configuration and which must remain server-side secrets.

Do not add a mock shape that the backend contract does not define. Module 10 introduces the shared [backend contract](../capstone/backend-contract.md) and MSW handlers.

### Conceptual checkpoint

Answer these in your own words:

1. When you call `Array.prototype.sort`, what object changes?
2. Does `as Product[]` inspect JSON at runtime? Why not?
3. Why does `fetch` need a status check even if the Promise resolved?
4. When should a shared value be in a component-local variable, a React state value, a URL, a Query cache, or Zustand?
5. Which step type-checks the app, and which step creates the production bundle?

---

## 8. Module 1 closeout

### Summary

- JavaScript is the runtime language; React describes UI; React DOM updates browser DOM; TypeScript checks source; Vite serves/transforms/builds assets.
- Use pure transformations and avoid mutating shared input collections.
- `async`/`await` is Promise syntax, not a way to run CPU work on a new thread.
- `fetch` rejects on network/abort problems, not automatically on HTTP error status.
- TypeScript types do not validate remote JSON. Parse trust-boundary values at runtime.
- Commit a lockfile, pin the selected stable direct dependencies, and run typecheck/lint/format/build gates.
- Give a component a job and a stable key; use semantic HTML and responsive CSS from the beginning.

### Mental model

```text
External inputs are untrusted
       ↓ validate/parse at a boundary
Domain values with explicit types
       ↓ pass through props/composition
Pure render describes the current interface
       ↓ React reconciles and React DOM commits changes
Browser presents the interface; user events may request new state
```

### Common mistakes

- Treating React, TypeScript, and Vite as the same tool.
- Assuming TypeScript's types exist in the deployed browser bundle.
- Mutating an array/object and expecting identity-based UI logic to infer what changed.
- Using array indexes as list identities.
- Forgetting HTTP status handling because `fetch` resolved.
- Putting API secrets in `.env` variables prefixed with `VITE_`.
- Installing every library at project creation before knowing its responsibility.
- Disabling Strict Mode, lint rules, or TypeScript strictness to silence symptoms.
- Treating a running dev server as proof that the production build works.

### Production tips

- Keep app feature data and code together until there is a real shared boundary.
- Use exact direct dependencies and commit the generated lockfile; update in small reviewed changes.
- Keep public environment configuration distinct from server secrets. A client bundle is inspectable by users.
- Inspect the Network panel, not only the rendered UI; check status, payload, cookies, and cancellation.
- Keep currency, dates, pagination, sorting, and error semantics explicit in API contracts.
- Accessibility is cheaper to preserve than to retrofit: use headings/landmarks, real link/button elements, visible focus, and non-color-only status from the first screen.
- Browser code should use relative `/api` URLs; proxy/route on the server side for development and deployment.

### Official documentation for this module

- [React: Quick Start](https://react.dev/learn)
- [React: Render and Commit](https://react.dev/learn/render-and-commit)
- [React: Writing Markup with JSX](https://react.dev/learn/writing-markup-with-jsx)
- [React: Importing and Exporting Components](https://react.dev/learn/importing-and-exporting-components)
- [React: Rendering Lists and Keys](https://react.dev/learn/rendering-lists)
- [React: TypeScript](https://react.dev/learn/typescript)
- [React: You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
- [Vite: Getting Started](https://vite.dev/guide/)
- [Vite: TypeScript transformation vs type checking](https://vite.dev/guide/features#typescript)
- [Vite: Environment variables and modes](https://vite.dev/guide/env-and-mode)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript 7.0 announcement and compatibility changes](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/)
- [ESLint flat configuration](https://eslint.org/docs/latest/use/configure/configuration-files)
- [React Hooks ESLint plugin](https://react.dev/reference/eslint-plugin-react-hooks)
- [Prettier documentation](https://prettier.io/docs/)
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
- [Node.js release status](https://nodejs.org/en/about/previous-releases)
- [Git documentation](https://git-scm.com/docs)

### What you should build now

Create the project and implement the catalogue from the challenge. Run `npm run typecheck`, `npm run lint`, `npm run format:check`, and `npm run build`; open the page at mobile and desktop widths; navigate it with a keyboard; inspect the browser console and network panel. Save a Git checkpoint after the project setup and another after the catalogue.

### What you should know before moving forward

- Can you describe the boundary between runtime UI and build-time typechecking?
- Can you add a product without copying markup or widening the domain types to `any`?
- Can you explain why a type assertion is not a validator?
- Can you explain how an aborted `fetch` differs from a server authorization denial?
- Can you run, read, and fix each quality-gate failure without disabling the check?

**Do not move on until you can build this small page from an empty Vite starter and explain each file's responsibility.**
