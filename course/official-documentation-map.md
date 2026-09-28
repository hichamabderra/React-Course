# Official Documentation Map

> **Course verification snapshot: 2026-09-28.** This is a reading index, not a substitute for the lesson. Package versions and compatibility decisions are recorded in [Verified Stack](verified-stack.md); this page prioritizes official API authorities. Project documentation is continuously updated, so a link marked “latest” may show a version newer than the course pin. Before copying an API into the capstone, check the package version/compatibility matrix and the current release notes.

The syllabus is taught in phases. Read the few links attached to the current module first; use the rest when the relevant question appears. Each future module will include a smaller focused reading list.

## Phase 1 — Foundations and the React mental model

### Module 1 — Project, Git, JavaScript, and TypeScript bridge

**Read for the module:**

- [React Learn](https://react.dev/learn) — the official learning path and vocabulary.
- [React: Render and Commit](https://react.dev/learn/render-and-commit) — render calculation versus DOM commit.
- [Vite Guide](https://vite.dev/guide/) and [Vite features: TypeScript](https://vite.dev/guide/features#typescript) — Vite transforms TypeScript syntax but does not type-check the project.
- [Vite environment variables and modes](https://vite.dev/guide/env-and-mode) — public `VITE_*` values and build-time modes.
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) and [React with TypeScript](https://react.dev/learn/typescript) — static types, JSX, and component props.
- [Node.js release status](https://nodejs.org/en/about/previous-releases) — choose an actively maintained runtime line.

**Look up as needed:** [ESLint flat configuration](https://eslint.org/docs/latest/use/configure/configuration-files), [React Hooks ESLint plugin](https://react.dev/reference/eslint-plugin-react-hooks), [typescript-eslint dependency versions](https://typescript-eslint.io/users/dependency-versions/), [Prettier](https://prettier.io/docs/), [MDN Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), [MDN AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController), [Git reference](https://git-scm.com/docs), and [TypeScript 7.0 release announcement](https://devblogs.microsoft.com/typescript/announcing-typescript-7-0/).

The TS 6.0.3 starter pin is a tooling-compatibility decision with `typescript-eslint`, not a claim that TS 6 is the newest compiler. Recheck the documented supported range before changing it.

### Module 2 — Components, JSX, props, rendering, and state

- [Your First Component](https://react.dev/learn/your-first-component)
- [Importing and Exporting Components](https://react.dev/learn/importing-and-exporting-components)
- [Writing Markup with JSX](https://react.dev/learn/writing-markup-with-jsx)
- [JavaScript in JSX with Curly Braces](https://react.dev/learn/javascript-in-jsx-with-curly-braces)
- [Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component)
- [Conditional Rendering](https://react.dev/learn/conditional-rendering)
- [Rendering Lists and Keys](https://react.dev/learn/rendering-lists)
- [Responding to Events](https://react.dev/learn/responding-to-events)
- [State: A Component's Memory](https://react.dev/learn/state-a-components-memory)
- [State as a Snapshot](https://react.dev/learn/state-as-a-snapshot)
- [Queueing a Series of State Updates](https://react.dev/learn/queueing-a-series-of-state-updates)
- [React rules: components and Hooks must be pure](https://react.dev/reference/rules/components-and-hooks-must-be-pure)

### Module 3 — Hooks and effects by purpose

- [Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks)
- [Separating Events from Effects](https://react.dev/learn/separating-events-from-effects)
- [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
- [Lifecycle of Reactive Effects](https://react.dev/learn/lifecycle-of-reactive-effects)
- [Removing Effect Dependencies](https://react.dev/learn/removing-effect-dependencies)
- [Effect dependencies](https://react.dev/learn/lifecycle-of-reactive-effects#what-an-effect-with-empty-dependencies-means)
- [useEffect](https://react.dev/reference/react/useEffect), [useRef](https://react.dev/reference/react/useRef), [useMemo](https://react.dev/reference/react/useMemo), [useCallback](https://react.dev/reference/react/useCallback), [useContext](https://react.dev/reference/react/useContext), and [createContext](https://react.dev/reference/react/createContext)
- [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
- [StrictMode](https://react.dev/reference/react/StrictMode) and [the React Hooks ESLint plugin](https://react.dev/reference/eslint-plugin-react-hooks)

**Important:** `useEffectEvent` is for non-reactive event logic that genuinely belongs inside an Effect. It is not a device for hiding dependencies. Read [useEffectEvent](https://react.dev/reference/react/useEffectEvent) with its caveats.

### Module 4 — React 19.3, Actions, concurrent UI, Suspense, and Compiler

- [React 19.3 release notes](https://react.dev/blog/2026/09/09/react-19-3) — version-specific API and release context.
- [useActionState](https://react.dev/reference/react/useActionState), [useOptimistic](https://react.dev/reference/react/useOptimistic), [useTransition](https://react.dev/reference/react/useTransition), and [startTransition](https://react.dev/reference/react/startTransition) — Actions, pending state, and non-urgent UI work.
- [useDeferredValue](https://react.dev/reference/react/useDeferredValue) — defer rendering a value without confusing it with debouncing or network caching.
- [Suspense](https://react.dev/reference/react/Suspense), [lazy](https://react.dev/reference/react/lazy), and [Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary).
- [use](https://react.dev/reference/react/use) — promises/context and conditional reads; follow the caching and boundary requirements.
- [Activity](https://react.dev/reference/react/Activity) — preserve hidden UI state where that behavior is intentional.
- [ViewTransition](https://react.dev/reference/react/ViewTransition) — React-coordinated transitions; use only for a deliberate experience and verify framework/browser support.
- [Fragment](https://react.dev/reference/react/Fragment) — keyed fragments and the newer fragment-ref capability; do not add a wrapper only to obtain a ref.
- [React Compiler](https://react.dev/learn/react-compiler), [Compiler setup](https://react.dev/learn/react-compiler/installation), and [React Compiler 1.0 announcement](https://react.dev/blog/2025/10/07/react-compiler-1).
- [React Server Components](https://react.dev/reference/rsc/server-components) and [Server Functions](https://react.dev/reference/rsc/server-functions) — learn the architecture boundary; do not hand-build an RSC bundler for this Vite SPA.

The course teaches React Compiler diagnostics and profiling before enabling compiler transforms. Automatic optimization does not eliminate the need for correct state boundaries, pure rendering, or measurement.

## Phase 2 — Product structure, CSS, and navigation

### Module 5 — Architecture and component engineering

- [Thinking in React](https://react.dev/learn/thinking-in-react) — derive a component/data-flow model from a UI.
- [Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure) — avoid redundant and contradictory state.
- [Sharing State Between Components](https://react.dev/learn/sharing-state-between-components) — lift only to the common owner that needs it.
- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context) — use Context for appropriate cross-tree dependencies, not as a default remote cache.
- [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) — share behavior without forcing component inheritance or premature abstractions.
- [React component API guidance](https://react.dev/learn/passing-props-to-a-component) — props and composition are the public surface.

### Module 6 — CSS, Tailwind v4, responsive layout, and themes

- [Tailwind CSS with Vite](https://tailwindcss.com/docs/installation/using-vite) and [Tailwind v4.3 release](https://tailwindcss.com/blog/tailwindcss-v4-3) — current Vite integration and CSS-first configuration.
- [Theme variables](https://tailwindcss.com/docs/theme) — design tokens and generated utilities.
- [Responsive design](https://tailwindcss.com/docs/responsive-design) — mobile-first breakpoints.
- [Dark mode](https://tailwindcss.com/docs/dark-mode) — system preference and explicit theme selector patterns.
- [Hover, focus, and other states](https://tailwindcss.com/docs/hover-focus-and-other-states) — focus-visible, disabled, reduced-motion, and state variants.
- [MDN CSS Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout), [Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout), and [Media Queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries) — platform layout foundations.

### Module 7 — shadcn/ui, Base UI, and accessible primitives

- [shadcn/ui Vite setup](https://ui.shadcn.com/docs/installation/vite), [CLI](https://ui.shadcn.com/docs/cli), [theming](https://ui.shadcn.com/docs/theming), and [dark mode](https://ui.shadcn.com/docs/dark-mode) — generator workflow and source ownership.
- [shadcn components](https://ui.shadcn.com/docs/components) — reference implementation and component configuration.
- [Base UI React](https://base-ui.com/react) and its [component catalog](https://base-ui.com/react/components) — current default primitive source for new shadcn projects.
- [shadcn changelog](https://ui.shadcn.com/docs/changelog) — verify which primitive base new projects currently select; Radix remains a supported option.
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/) — interaction patterns for complex widgets; prefer native HTML where possible.

### Module 8 — React Router 8 Data Mode and URL design

- [React Router modes](https://reactrouter.com/start/modes) and [Data Mode installation](https://reactrouter.com/start/data/installation) — the capstone's Vite SPA choice.
- [Routing](https://reactrouter.com/start/data/routing), [Route Objects](https://reactrouter.com/start/data/route-object), [Data Loading](https://reactrouter.com/start/data/data-loading), and [Actions](https://reactrouter.com/start/data/actions).
- [Pending UI for Data Mode](https://reactrouter.com/start/data/pending-ui) — the guide points to shared pending-state patterns; prefer the API reference for the mode used by the app.
- [createBrowserRouter](https://reactrouter.com/api/data-routers/createBrowserRouter), [RouterProvider](https://reactrouter.com/api/data-routers/RouterProvider), [useNavigation](https://reactrouter.com/api/hooks/useNavigation), [useSearchParams](https://reactrouter.com/api/hooks/useSearchParams), [useRouteError](https://reactrouter.com/api/hooks/useRouteError), and [useParams](https://reactrouter.com/api/hooks/useParams).
- [React Router v8 upgrade guidance](https://reactrouter.com/upgrading/v7) and [changelog](https://reactrouter.com/changelog) — particularly important when following older v6/v7 tutorials.

The course does not fetch the same entity independently in a router cache and a TanStack Query cache. The module compares loader orchestration and Query cache reuse so the project has one deliberate server-data ownership strategy.

## Phase 3 — State, data, identity, and forms

### Module 9 — State ownership, Zustand, and Redux Toolkit comparison

- [React: Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure), [Sharing State](https://react.dev/learn/sharing-state-between-components), and [Reducer + Context](https://react.dev/learn/scaling-up-with-reducer-and-context).
- [Zustand introduction](https://zustand.docs.pmnd.rs/getting-started/introduction), [immutable state and merging](https://zustand.docs.pmnd.rs/guides/immutable-state-and-merging), [preventing rerenders with `useShallow`](https://zustand.docs.pmnd.rs/guides/prevent-rerenders-with-use-shallow), and [Zustand v5 migration](https://zustand.docs.pmnd.rs/reference/migrations/migrating-to-v5).
- [Redux Toolkit getting started](https://redux-toolkit.js.org/introduction/getting-started), [Redux Essentials](https://redux.js.org/tutorials/essentials/part-1-overview-concepts), and [RTK Query overview](https://redux-toolkit.js.org/rtk-query/overview).

Use these references to choose a client-store boundary. Do not put URL filters, form fields, Query-owned server records, and local dialog state into one global store.

### Module 10 — REST API client and TanStack Query v5

- [TanStack Query React overview](https://tanstack.com/query/latest/docs/framework/react/overview), [important defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults), [queries](https://tanstack.com/query/latest/docs/framework/react/guides/queries), [query keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys), and [query functions](https://tanstack.com/query/latest/docs/framework/react/guides/query-functions).
- [Mutations](https://tanstack.com/query/latest/docs/framework/react/guides/mutations), [query invalidation](https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation), [updates from mutation responses](https://tanstack.com/query/latest/docs/framework/react/guides/updates-from-mutation-responses), and [optimistic updates](https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates).
- [Query cancellation](https://tanstack.com/query/latest/docs/framework/react/guides/query-cancellation), [paginated queries](https://tanstack.com/query/latest/docs/framework/react/guides/paginated-queries), [infinite queries](https://tanstack.com/query/latest/docs/framework/react/guides/infinite-queries), [dependent queries](https://tanstack.com/query/latest/docs/framework/react/guides/dependent-queries), [prefetching](https://tanstack.com/query/latest/docs/framework/react/guides/prefetching), and [testing](https://tanstack.com/query/latest/docs/framework/react/guides/testing).
- [TanStack Query v5 migration guide](https://tanstack.com/query/latest/docs/framework/react/guides/migrating-to-v5) — current object syntax and changed lifecycle options.
- [MDN Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), [HTTP response status](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status), and [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController) — browser network behavior.
- [Zod](https://zod.dev/) — later used to validate untrusted API data at runtime.
- [MSW documentation](https://mswjs.io/docs/) — realistic network-boundary mocks.

See [Northstar backend contract](capstone/backend-contract.md) for the capstone's endpoint, error, pagination, money, and session assumptions.

### Module 11 — Authentication, sessions, cookies, and security

- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html), [Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html), and [Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html).
- [MDN Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies), [Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie), [Fetch credentials](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials), and [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).
- [OAuth 2.0 for Browser-Based Apps (IETF draft/status page)](https://datatracker.ietf.org/doc/draft-ietf-oauth-browser-based-apps/) — consult current publication status and security guidance; do not rely on an outdated blog summary.
- [OWASP HTML5 Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTML5_Security_Cheat_Sheet.html) — browser storage and client-side trust boundaries.

The course's default capstone is an opaque HttpOnly session cookie. Short-lived in-memory access tokens plus an HttpOnly refresh cookie are compared as an alternate deployment model. Local-storage bearer-token persistence is not the default.

### Module 12 — Authorization and RBAC

- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html) and [OWASP IDOR Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html).
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/) — object-level authorization, function-level authorization, and excessive data exposure risks.
- [MDN HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status) — distinguish 401 and 403.
- [React Router route objects](https://reactrouter.com/start/data/route-object) — navigation/UI boundaries only; not a security enforcement mechanism.

### Module 13 — Forms, RHF, Zod, and server errors

- [React Hook Form `useForm`](https://react-hook-form.com/docs/useform), [`register`](https://react-hook-form.com/docs/useform/register), [`formState`](https://react-hook-form.com/docs/useform/formstate), [`Controller`](https://react-hook-form.com/docs/usecontroller/controller), and [`useFieldArray`](https://react-hook-form.com/docs/usefieldarray).
- [React Hook Form resolvers — Zod integration](https://github.com/react-hook-form/resolvers#zod) — resolver inputs/outputs and schema integration.
- [Zod documentation](https://zod.dev/) and [Zod API](https://zod.dev/api) — runtime parsing, safe parsing, refinements, transformations, and inferred input/output types.
- [React: `<form>`](https://react.dev/reference/react-dom/components/form), [useActionState](https://react.dev/reference/react/useActionState), and [useFormStatus](https://react.dev/reference/react-dom/hooks/useFormStatus) — native React form Actions and pending state; compare carefully with the capstone's RHF/API contract rather than mixing patterns without a reason.
- [WAI forms tutorial](https://www.w3.org/WAI/tutorials/forms/) and [WCAG form labels and instructions](https://www.w3.org/WAI/WCAG22/Understanding/labels-or-instructions.html).

## Phase 4 — Admin UX, accessibility, and motion

### Module 14 — Admin tables and resilient UI states

- [TanStack Table v9 React overview](https://tanstack.com/table/latest/docs/framework/react/overview), [Quick Start](https://tanstack.com/table/latest/docs/framework/react/quick-start), and [migration to v9](https://tanstack.com/table/latest/docs/framework/react/guide/migrating).
- [Sorting](https://tanstack.com/table/latest/docs/framework/react/guide/sorting), [column filtering](https://tanstack.com/table/latest/docs/framework/react/guide/column-filtering), [pagination](https://tanstack.com/table/latest/docs/framework/react/guide/pagination), and [table state](https://tanstack.com/table/latest/docs/framework/react/guide/table-state).
- [TanStack Virtual React introduction](https://tanstack.com/virtual/latest/docs/framework/react/introduction) — only consider after measuring real DOM/list costs.
- [React Router pending UI](https://reactrouter.com/start/framework/pending-ui) and [TanStack Query background fetching indicators](https://tanstack.com/query/latest/docs/framework/react/guides/background-fetching-indicators) — choose pending states appropriate to the owning data layer.
- [WAI-ARIA table pattern](https://www.w3.org/WAI/ARIA/apg/patterns/table/) — use semantic HTML tables for static tabular information and follow the pattern only when implementing a composite interactive grid.

**Version warning:** the course selects TanStack Table 9.2.4. V9 uses `useTable` and explicit feature registration; do not paste v8's `useReactTable` examples without consciously choosing the legacy API path.

### Module 15 — Accessibility engineering

- [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/) and [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/).
- [WAI-ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) and [WAI tutorials](https://www.w3.org/WAI/tutorials/) — patterns and implementation guidance.
- [Accessible Name and Description Computation](https://www.w3.org/TR/accname-1.2/), [WAI keyboard interface guidance](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/), and [MDN ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA).
- [Playwright accessibility testing](https://playwright.dev/docs/accessibility-testing) — automated checks catch only some problems; keep manual review.
- [Storybook accessibility testing](https://storybook.js.org/docs/writing-tests/accessibility-testing) — useful for isolated component states when Storybook is adopted.

### Module 16 — Motion and reduced-motion behavior

- [Motion for React](https://motion.dev/docs/react), [animation](https://motion.dev/docs/react-animation), [AnimatePresence](https://motion.dev/docs/react-animate-presence), [layout animations](https://motion.dev/docs/react-layout-animations), [gestures](https://motion.dev/docs/react-gestures), and [accessibility](https://motion.dev/docs/react-accessibility).
- [Motion `useReducedMotion`](https://motion.dev/docs/react-use-reduced-motion) — adapt nonessential animation to user preference.
- [React ViewTransition](https://react.dev/reference/react/ViewTransition) — browser View Transitions coordinated through React where supported.
- [MDN `prefers-reduced-motion`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion) and [WCAG Animation from Interactions](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html).

Prefer CSS transitions for simple state styling. Motion is justified for orchestration, gesture, enter/exit, or layout behavior that would otherwise be materially harder to manage. The course pins the current `motion` package and imports React APIs from `motion/react`; some otherwise-current Motion accessibility examples still show the older `framer-motion` import, so translate imports to the pinned package and recheck its changelog rather than copying that line verbatim.

## Phase 5 — Testing, performance, security, and delivery

### Module 17 — Testing strategy

- [Vitest Guide](https://vitest.dev/guide/), [API](https://vitest.dev/api/), [mocking](https://vitest.dev/guide/mocking), [projects](https://vitest.dev/guide/projects), and [migration guide](https://vitest.dev/guide/migration/).
- [jsdom on npm](https://www.npmjs.com/package/jsdom) — the course pins **30.1.0** for its Node-based DOM tests; verify its Node engine requirement and remember it is not a real browser.
- [React Testing Library introduction](https://testing-library.com/docs/react-testing-library/intro/), [queries](https://testing-library.com/docs/queries/about/), [user-event](https://testing-library.com/docs/user-event/intro/), and [async methods](https://testing-library.com/docs/dom-testing-library/api-async/).
- [MSW](https://mswjs.io/docs/) — shared HTTP-level network handlers in tests/development.
- [Playwright Test](https://playwright.dev/docs/intro), [best practices](https://playwright.dev/docs/best-practices), [network](https://playwright.dev/docs/network), [authentication](https://playwright.dev/docs/auth), and [trace viewer](https://playwright.dev/docs/trace-viewer-intro).
- [TanStack Query testing guidance](https://tanstack.com/query/latest/docs/framework/react/guides/testing) — isolate clients and avoid cross-test cache leakage.

Do not assert internal hook calls or implementation-specific component state when user-visible behavior will do. Use unit tests for pure logic, Testing Library for component/integration behavior, MSW for HTTP contract cases, and Playwright for critical browser journeys.

### Module 18 — Storybook and component collaboration

- [Storybook for React + Vite](https://storybook.js.org/docs/get-started/frameworks/react-vite), [writing stories](https://storybook.js.org/docs/writing-stories), and [component documentation](https://storybook.js.org/docs/writing-docs).
- [Storybook interaction tests](https://storybook.js.org/docs/writing-tests/interaction-testing), [accessibility tests](https://storybook.js.org/docs/writing-tests/accessibility-testing), [Vitest addon](https://storybook.js.org/docs/writing-tests/integrations/vitest-addon), and [Storybook 10 migration guide](https://storybook.js.org/docs/releases/migration-guide).

Storybook is an optional collaboration/tooling layer. Add it when the team benefits from isolated stateful components; do not duplicate the app into a story-only architecture.

### Module 19 — Performance, profiling, and React Compiler

- [React Developer Tools](https://react.dev/learn/react-developer-tools) and [React Profiler API](https://react.dev/reference/react/Profiler) — inspect render work before memoizing.
- [React Compiler](https://react.dev/learn/react-compiler), [installation](https://react.dev/learn/react-compiler/installation), and [eslint-plugin-react-hooks compiler diagnostics](https://react.dev/reference/eslint-plugin-react-hooks).
- [Vite build guide](https://vite.dev/guide/build) and [dynamic import/code splitting](https://vite.dev/guide/features#dynamic-import) — inspect route chunks and production output.
- [web.dev: Core Web Vitals](https://web.dev/articles/vitals) — current field metrics and thresholds; record the date/profile of any benchmark.
- [web.dev: image performance](https://web.dev/learn/images/) and [MDN lazy loading](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/Lazy_loading).
- [TanStack Virtual](https://tanstack.com/virtual/latest/docs/framework/react/introduction) — use only if profiling supports the complexity.

The optimization sequence is: define the journey and budget → measure → identify the bottleneck → change one cause → measure again. `memo`, `useMemo`, and `useCallback` are not default decorations.

### Module 20 — Production quality, security, operations, and deployment

- [OWASP Top 10](https://owasp.org/www-project-top-ten/), [OWASP Cross-Site Scripting Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html), [Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html), and [OWASP Dependency Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Dependency_Management_Cheat_Sheet.html).
- [MDN Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP), [Trusted Types API](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API), [React 19.3 Trusted Types announcement](https://react.dev/blog/2026/09/09/react-19-3), and [React `dangerouslySetInnerHTML`](https://react.dev/reference/react-dom/components/common#dangerously-setting-the-inner-html).
- [Vite static deployment](https://vite.dev/guide/static-deploy) and [environment configuration](https://vite.dev/guide/env-and-mode) — SPA fallback routing, output path, and public configuration.
- [React error boundary guidance](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary), [TanStack Query retry defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults), and [Playwright CI](https://playwright.dev/docs/ci-intro).
- [GitHub Actions documentation](https://docs.github.com/en/actions) — if using GitHub-hosted CI; never put untrusted secrets in client build variables.
- [OWASP Session](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html), [CSRF](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html), [Authorization](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html), and [CORS (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) — revisit server trust boundaries at release time.

### Module 21 — Northstar independent delivery and architecture review

- [Northstar backend contract](capstone/backend-contract.md) — the API/auth assumptions the frontend must either use or challenge explicitly.
- [Final challenge PRD](capstone/final-challenge-prd.md) — requirements only; submit an architecture proposal before writing implementation.
- Revisit the specific docs linked by your chosen state/data/router/testing design. Include the version and source for every library/API decision in your proposal.
- Use [GitHub pull request review guidance](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests) for a self-review/reviewer checklist if delivery is through GitHub.

## Topic index

For cross-cutting work, jump directly to the official source families:

- **React fundamentals / hooks:** [React Learn](https://react.dev/learn) · [API Reference](https://react.dev/reference/react) · [Rules](https://react.dev/reference/rules)
- **Build/runtime:** [Vite](https://vite.dev/guide/) · [Node releases](https://nodejs.org/en/about/previous-releases) · [TypeScript](https://www.typescriptlang.org/docs/)
- **Navigation:** [React Router](https://reactrouter.com/start/modes)
- **Async server data:** [TanStack Query](https://tanstack.com/query/latest/docs/framework/react/overview) · [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) · [MSW](https://mswjs.io/docs/)
- **Client state:** [Zustand](https://zustand.docs.pmnd.rs/) · [Redux Toolkit](https://redux-toolkit.js.org/)
- **Styling/UI:** [Tailwind CSS](https://tailwindcss.com/docs) · [shadcn/ui](https://ui.shadcn.com/docs) · [Base UI](https://base-ui.com/react)
- **Forms and validation:** [React Hook Form](https://react-hook-form.com/docs) · [Zod](https://zod.dev/)
- **Testing:** [Vitest](https://vitest.dev/guide/) · [Testing Library](https://testing-library.com/docs/react-testing-library/intro/) · [Playwright](https://playwright.dev/docs/intro)
- **Accessibility:** [WCAG 2.2](https://www.w3.org/TR/WCAG22/) · [WAI-ARIA APG](https://www.w3.org/WAI/ARIA/apg/)
- **Security:** [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) · [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)
