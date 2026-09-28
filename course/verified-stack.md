# Verified Stack and Selection Notes

> **Verification date: 2026-09-28.** Stable versions were checked against current official project documentation/release notes and each package's npm `latest` tag. The API references below—not a tutorial written for an older major version—are the teaching authority. Package tags continue to move; before starting a later library module, re-check the project's official changelog, compatibility requirements, and package metadata, then commit the lockfile.

## Verified versions

The full stack is intentionally layered. The first project checkpoint installs only React, Vite, TypeScript, and quality tooling; data, UI, animation, and testing packages are added when their problem is introduced.

| Layer | Stable package/version used or considered | Official docs / release evidence | Scope in this course |
|---|---|---|---|
| Runtime | Node.js **24.21.0 LTS** | [Node releases](https://nodejs.org/en/about/previous-releases) · [v24.21.0 release](https://nodejs.org/en/blog/release/v24.21.0) | Course runtime. Vite 8 and React Router 8 have lower version floors; Node 24 LTS satisfies both. |
| React | `react` **19.3.0**, `react-dom` **19.3.0** | [React version history](https://react.dev/versions) · [React 19.3 release](https://react.dev/blog/2026/09/09/react-19-3) | Rendering, DOM integration, hooks, Actions, Suspense, modern refs, `Activity`, View Transitions, Fragment refs, and Trusted Types integration. |
| Vite | `vite` **8.3.1**, `@vitejs/plugin-react` **6.1.1** | [Vite guide](https://vite.dev/guide/) · [Vite 8 release](https://vite.dev/blog/announcing-vite8) · [Vite React plugin](https://github.com/vitejs/vite-plugin-react) | Development server, transformations, HMR, build, and deploy. Vite 8 is Rolldown-powered; Vite is not a React framework or API server. |
| TypeScript | **6.0.3 course baseline**; latest stable compiler is **7.0.2** | [TypeScript 7.0 release](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/) · [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) | TS6.0.3 is selected for the combined compiler + current TypeScript-aware ESLint baseline. TS7 is covered as a current ecosystem transition, not silently ignored. |
| React types / Node types | `@types/react` **19.3.0**, `@types/react-dom` **19.3.0**, `@types/node` **24.19.0** | [React TypeScript guide](https://react.dev/learn/typescript) · [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) | Module 1's pinned starter. `@types/node` follows the chosen Node 24 line. |
| Router | `react-router` **8.4.0** | [Modes](https://reactrouter.com/start/modes) · [Data Mode installation](https://reactrouter.com/start/data/installation) · [v8 changelog](https://reactrouter.com/changelog) | Data Mode for the capstone's Vite SPA. Use `react-router` for most APIs and `react-router/dom` for the DOM `RouterProvider`. |
| Server state | `@tanstack/react-query` **5.104.0** | [React Query overview](https://tanstack.com/query/latest/docs/framework/react/overview) · [v5 migration guide](https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5) | Queries, mutations, keys, freshness, cache lifecycle, retries, optimistic updates, pagination, cancellation, and prefetching. |
| Client state | `zustand` **5.0.15** | [Zustand docs](https://zustand.docs.pmnd.rs/) · [v5 migration notes](https://zustand.docs.pmnd.rs/reference/migrations/migrating-to-v5) | One primary client-state library. Applied to a small guest-cart draft; not server data, forms, or route filters. |
| State alternative | `@reduxjs/toolkit` **2.12.0** | [Redux Toolkit docs](https://redux-toolkit.js.org/) · [RTK 2 migration](https://redux-toolkit.js.org/usage/migrating-rtk-2) | Compared for larger teams and workflows that benefit from explicit reducer/event conventions. Not added as a second app store. |
| Styling | `tailwindcss` **4.3.3**, `@tailwindcss/vite` **4.3.3** | [Tailwind + Vite](https://tailwindcss.com/docs/installation/using-vite) · [Tailwind 4.3 release](https://tailwindcss.com/blog/tailwindcss-v4-3) | CSS-first tokens/utilities with plain CSS and CSS Modules where they improve boundaries. |
| UI primitives/source | `shadcn` CLI **4.21.0**, `@base-ui/react` **1.8.0** | [shadcn Vite setup](https://ui.shadcn.com/docs/installation/vite) · [Base UI](https://base-ui.com/react) · [Base UI default announcement](https://ui.shadcn.com/docs/changelog/2026-07-base-ui-default) | shadcn generates source into the app; Base UI is the current default primitive base. Radix remains supported. |
| Forms | `react-hook-form` **7.89.0**, `@hookform/resolvers` **5.9.1** | [RHF `useForm`](https://react-hook-form.com/docs/useform) · [Resolvers](https://github.com/react-hook-form/resolvers) | Registration, controlled adapters when necessary, field subscriptions, submission state, and server errors. RHF v8 is not used as the stable baseline here. |
| Runtime schemas | `zod` **4.6.5** | [Zod docs](https://zod.dev/) · [Zod 4.6 announcement](https://zod.dev/blog/zod-4-6) | Runtime validation and type inference at API/form boundaries. Zod 4 is stable. |
| Animation | `motion` **13.4.4** | [Motion for React](https://motion.dev/docs/react) · [Motion changelog](https://motion.dev/changelog) | Import from `motion/react`; use only where motion improves understanding or feedback. |
| Test runner | `vitest` **5.0.2**, `@vitest/coverage-v8` **5.0.2** | [Vitest 5 release](https://vitest.dev/blog/vitest-5.html) · [Vitest guide](https://vitest.dev/guide/) | Unit and integration/component test orchestration. Vitest 5 requires Node `>=22.12.0` and Vite `>=6.4.0`. |
| React test helpers / DOM environment | `@testing-library/react` **16.3.3**, `@testing-library/user-event` **14.6.7**, `@testing-library/dom` **10.4.2**, `@testing-library/jest-dom` **7.0.1**, `jsdom` **30.1.0** | [RTL introduction](https://testing-library.com/docs/react-testing-library/intro/) · [user-event](https://testing-library.com/docs/user-event/intro/) · [jsdom on npm](https://www.npmjs.com/package/jsdom) | Test the interface the way a person uses it: accessible queries, keyboard and pointer interactions, async outcomes. jsdom supplies a simulated DOM; it is not a real browser. |
| Browser E2E | `@playwright/test` **1.63.0** | [Playwright Test](https://playwright.dev/docs/intro) · [release notes](https://playwright.dev/docs/release-notes) | Cross-browser user journeys, permissions, cookies, navigation, and production build smoke tests. |
| Network mocks | `msw` **2.15.0** | [MSW docs](https://mswjs.io/docs/) · [MSW 2 announcement](https://mswjs.io/blog/introducing-msw-2.0/) | Reuse HTTP handlers in browser development and Node tests; intercept real `fetch` requests at the network boundary. |
| Linting | `eslint` **10.11.0**, `@eslint/js` **10.0.1**, `typescript-eslint` **8.70.1**, `eslint-plugin-react-hooks` **7.1.1**, `eslint-plugin-react-refresh` **0.5.7** | [ESLint config](https://eslint.org/docs/latest/use/configure/configuration-files) · [React Hooks ESLint plugin](https://react.dev/reference/eslint-plugin-react-hooks) · [typescript-eslint support](https://typescript-eslint.io/users/dependency-versions/) | ESLint flat config, React rules, compiler diagnostics, and TS syntax/type-aware rules compatible with the pinned TS6 baseline. |
| Formatting | `prettier` **3.9.9** | [Prettier docs](https://prettier.io/docs/) | Formatting only; ESLint finds likely defects and policy violations. Keep the jobs separate. |
| React Compiler | `babel-plugin-react-compiler` **1.0.0** | [React Compiler docs](https://react.dev/learn/react-compiler) · [v1.0 announcement](https://react.dev/blog/2025/10/07/react-compiler-1) · [Vite 8 release](https://vite.dev/blog/announcing-vite8) | Introduced after the render model and profiling. Compiler diagnostics are useful even before enabling compilation. |
| Isolated component workshop | `storybook` **10.6.0** | [Storybook docs](https://storybook.js.org/docs) · [release posts](https://storybook.js.org/blog/tag/release/) | Optional but included later for shared components, visual states, interaction tests, and documentation. |
| Admin tables | `@tanstack/react-table` **9.2.4** | [Table v9 overview](https://tanstack.com/table/latest/docs/framework/react/overview) · [v9 migration](https://tanstack.com/table/latest/docs/framework/react/guide/migrating) | Headless data-table logic when native table markup alone is not enough. Added for admin use, not every small list. |
| Long-list virtualization | `@tanstack/react-virtual` **3.14.13** | [Virtual React docs](https://tanstack.com/virtual/latest/docs/framework/react/introduction) | Add only after profiling shows DOM/render volume is a real cost. |
| Icons | `lucide-react` **1.48.0** | [Lucide React](https://lucide.dev/guide/packages/lucide-react) | A consistent icon set; decorative icons are hidden from assistive technology, functional icon-only buttons receive accessible names. |

### Version evidence and reproducibility

Exact patch versions above are a **2026-09-28 snapshot**, not a promise that a future install of `@latest` resolves to the same patch. In Module 1 we use `--save-exact` and commit `package-lock.json`. During later modules, install a dependency when it becomes necessary, review the official changelog and peer requirements, then record the lockfile change. Do not copy a version number from an old blog or float `latest` in a production build.

The principal browser-runtime floors checked for this path are:

- **Vite 8:** Node.js `20.19+` or `22.12+`; see [Vite 8 release notes](https://vite.dev/blog/announcing-vite8).
- **React Router 8:** Node.js `22.22+`, React and React DOM `19.2.7+`; see [React Router's upgrade guide](https://reactrouter.com/upgrading/v7).
- **Vitest 5:** Node.js `22.12+`, Vite `6.4+`; see [Vitest's migration guide](https://vitest.dev/guide/migration/).

Node 24.21.0 LTS and React 19.3.0 meet the relevant minima.

## Why these selections (and when not to use them)

### React and Vite instead of starting with a meta-framework

The capstone is a browser-rendered commerce/admin client backed by a separately specified HTTP API. A Vite SPA makes the UI/runtime/build boundary explicit and fits the requested stack. The course still teaches how to evaluate React Router Framework Mode or a full-stack framework when requirements include server rendering, server data access, progressive enhancement, or server functions. A SPA is not automatically the correct choice for every public, SEO-sensitive product page.

### React Router Data Mode and TanStack Query have separate jobs

React Router owns navigation, matched route/layout state, URL transitions, route-level pending/error presentation, and optional route orchestration. TanStack Query owns reusable server-data cache lifecycle. For a route loader that needs products, the planned pattern is to call `queryClient.ensureQueryData(productOptions)` and render the same key through `useQuery`; avoid fetching into one loader cache and then duplicating the same entity in another independent cache. If a product uses only TanStack Query and does not need route loader semantics, Declarative Mode can be simpler. If the app wants the routing/build/deploy framework to own more, Framework Mode is an alternative.

### Zustand first; Redux Toolkit as a considered alternative

Local state and Context are enough for many apps. Zustand is selected for one genuinely cross-route client-only state case with compact store code and selective subscriptions. The store is not a dumping ground. Redux Toolkit is appropriate when many teams benefit from standardized actions/reducers, explicit event history, middleware, established tooling, or RTK Query as the chosen server-data layer. If choosing RTK Query, reconsider whether a separate TanStack Query cache is worth the duplicated concepts. Do not place the same server entities in Query and Redux/Zustand without an explicit synchronization reason.

Zustand v5 also requires stable selector outputs under its default `Object.is` equality. Avoid allocating a fresh object from a selector on every render; select fields individually or use the documented `useShallow` helper.

### `fetch` first; no API-client package by default

Native `fetch` supports `Request`, `Response`, `AbortSignal`, cookies, and browser networking without another dependency. A small application client can centralize base paths, credentials, timeout/cancellation, error normalization, and runtime parsing. `fetch` does **not** reject on HTTP 4xx/5xx, so the client must inspect `response.ok`. Axios is reasonable when a team needs its adapter ecosystem, established interceptors, upload progress, or has a strong existing convention; it is not needed just to send JSON.

REST is used for the project contract because it makes HTTP semantics and cache invalidation concrete. GraphQL can be a better choice for clients needing flexible graph-shaped reads or generated operation contracts, but it does not remove authorization, runtime validation, error handling, or cache design.

### Tailwind v4, CSS Modules, plain CSS, and inline styles

Tailwind's CSS-first tokens and utilities are the primary app styling workflow, with ordinary CSS for global tokens, complex selectors, and platform features. CSS Modules are useful for a component's encapsulated styles or a team that prefers class-local CSS. Plain CSS is a good choice for global layout, keyframes, and small projects. Inline styles are useful for truly dynamic values (for example a runtime-calculated progress width), not as the default styling system: they do not replace pseudo-classes, media queries, token governance, or a maintainable variant API. Component-library styles are inspected and customized, not assumed to be immutable.

### shadcn/ui + Base UI; Radix remains a valid choice

shadcn/ui is a **source generator/registry workflow**, not a closed runtime component package. The copied component source belongs to the application and can be reviewed, themed, and maintained. As of the July 2026 shadcn changelog, new projects default to **Base UI**; **Radix is not deprecated** and continues to be supported. React Aria is also a supported base as of July 2026. We select Base UI because it is the current default path and has an official React component library. An existing Radix project does not need migration merely because the default changed.

Use semantic native elements when they solve the problem. A headless primitive is valuable for a complex composite widget (dialog, menu, combobox) where focus and keyboard interaction are substantial behavior—not for a styled paragraph.

### Storybook, Table, Virtual, icons and dates

| Tool | Why we include or evaluate it | When not to use it |
|---|---|---|
| **MSW 2.15.0** | Simulates realistic HTTP responses at the network boundary for development, integration tests, and browser tests. The app still runs its actual `fetch` code. | Do not use it to hide a broken contract or substitute for a few high-value tests against a real staging API. |
| **TanStack Table v9.2.4** | Headless state, sorting, filtering, selection, pagination, and column behavior for the admin grid while the team keeps control of accessible semantic markup and styling. | Do not add it for a simple static table or use client-side sorting for data that the server must authorize/page. |
| **TanStack Virtual v3.14.13** | Keeps thousands of rows/items from becoming thousands of simultaneous DOM nodes. | Do not virtualize a short list without a measured problem; windowing can complicate focus, screen-reader navigation, row heights, and scroll behavior. |
| **Storybook 10.6.0** | Isolates components and makes their empty/loading/error/disabled/responsive/permission variants visible to engineers, designers, and QA. Includes docs and interaction/a11y tooling through addons. | Do not maintain a second “storybook-only” component API or duplicate every application page without a collaboration/testing need. |
| **Lucide React 1.48.0** | Consistent icons that pair naturally with the selected shadcn source setup. | Do not rely on an icon alone to name an action; do not import a large icon namespace if tree-shaking can be avoided. |
| **Date library: not in the baseline** | Use `Intl.DateTimeFormat` for display; it is built into the platform and handles locale formatting. Consider date-fns/Temporal tooling only when date arithmetic, recurrence, parsing, or time-zone rules justify a dependency. | Do not add a date package just to format one ISO timestamp. Never assume a calendar day is a fixed 24-hour duration across time zones/DST. |
| **Axe/accessibility automation** | Use browser/Storybook axe checks to catch some automated WCAG rule failures alongside keyboard and assistive-technology review. | Automated scans cannot prove a flow is accessible or replace manual focus, reading order, zoom, contrast, and screen-reader testing. |

## IMPORTANT: Older tutorials may show…

| Older approach | Modern course approach |
|---|---|
| React 17/18 root setup with `ReactDOM.render(...)` | React 19 `createRoot` from `react-dom/client`; `StrictMode` in development. |
| `forwardRef` as the only way to pass a ref through a new function component | In React 19, function components can accept `ref` as a prop. `forwardRef` remains relevant to existing libraries/compatibility; React documents the newer ref pattern. |
| Effects used to derive values or keep two pieces of React state in sync | Derive during render or update the correct owner in the originating event; use Effects to synchronize with an external system. See [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect). |
| “Concurrent mode” as a separate application mode or a special root to enable | Current React APIs such as transitions, Suspense, deferred values, Actions, and concurrent rendering behavior are taught as scheduling/rendering capabilities—not a second React mode to turn on. |
| `react-router-dom` imports in a new React Router 8 app | React Router 8 removed the `react-router-dom` re-export. Import most APIs from `react-router`; import DOM `RouterProvider`/`HydratedRouter` from `react-router/dom`. |
| TanStack Query v4 positional `useQuery(key, fn, options)`, `cacheTime`, `keepPreviousData`, or query callbacks | v5 object options, `gcTime`, `placeholderData`/`keepPreviousData` helper, and no `useQuery` success/error/settled callbacks. |
| Tailwind v3 `@tailwind base; @tailwind components; @tailwind utilities;` plus JavaScript config as the default | Tailwind v4 Vite plugin and CSS-first `@import "tailwindcss";` / `@theme` tokens. Legacy JS config can be loaded for migration, but is not the new-project path. |
| shadcn examples that imply new projects must use Radix | Base UI is the current new-project default; Radix is supported, and existing apps need not migrate. Inspect generated code and primitive APIs for the selected base. |
| `framer-motion` imports as the only Motion package entry point | Current Motion package `motion`, with React APIs imported from `motion/react`. |
| Vitest config using the old `workspace` option | Vitest 5 uses its current `projects` configuration. Consult the current migration guide before copying old configuration. |
| TanStack Table v8's `useReactTable` and assume-every-feature-is-included model | TanStack Table v9 uses `useTable`, explicit registered features, and a redesigned table state model. We teach v9 for new work. |
| TypeScript is described as runtime validation | TypeScript types are erased. Parse network/form data at runtime (Zod in this course) and treat the result as untrusted until it passes. |
| `VITE_*` is treated as a secret store | Every `VITE_*` value is bundled into client code and is public. Keep secrets on the server. |
| Long-lived bearer/refresh credentials are placed in `localStorage` by default | Capstone default is a Secure, HttpOnly, appropriately SameSite server session cookie. The direct SPA token variation keeps refresh credentials HttpOnly and access credentials short-lived/in-memory only, with CSRF/XSS trade-offs made explicit. |

### Server Components decision

React Server Components (RSC) and Server Functions are relevant modern React capabilities, but this course's capstone is a Vite SPA consuming a separately specified REST backend. We teach what RSC solves and when a full-stack framework is a better architecture, but we do **not** build a custom RSC bundler/server as a prerequisite to learning frontend engineering. React's own documentation distinguishes stable component behavior from the lower-level implementation APIs that a bundler/framework needs; those implementation APIs have not followed normal minor-version semver in React 19. Use an official framework/bundler integration when adopting RSC, and re-check its compatibility matrix then.

Official reference: [React Server Components](https://react.dev/reference/rsc/server-components) and [Server Functions](https://react.dev/reference/rsc/server-functions).

## Official documentation and source map

The consolidated reading map, including the APIs taught by section, lives at [Official Documentation Map](official-documentation-map.md). Major modules will also end with a focused reading list so learners do not have to absorb every reference at once.
