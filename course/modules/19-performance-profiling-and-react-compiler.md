# Module 19 — Performance, Profiling, and React Compiler

> **Build goal:** measure the product catalogue, admin table, and route bundle; select one real bottleneck; improve it; and report before/after evidence. Do not add `memo`, `useMemo`, `useCallback`, or virtualization merely because a page re-renders.

## 1. Define user-visible performance

Performance is not “minimum render count.” It includes loading time, interaction response, layout stability, smoothness, memory, and resilience on slow devices/networks. Define the journey and audience before choosing an optimization.

Current Core Web Vitals “good” targets are evaluated at the 75th percentile of page visits:

- **LCP:** no more than 2.5 seconds.
- **INP:** no more than 200 milliseconds.
- **CLS:** no more than 0.1.

Confirm the current official definitions/thresholds when measuring; page type, device/network mix, and attribution matter.

## 2. Profile before changing code

Use React Developer Tools Profiler and browser Performance/Network panels:

1. Reproduce a user journey with representative data/device throttling.
2. Record the interaction/route and the baseline.
3. Identify whether time is spent in JavaScript, React render, layout, image decode, network, or server response.
4. Change one cause.
5. Repeat the same measurement and confirm no accessibility/functional regression.

React can re-render a component without changing the DOM. A re-render is not automatically a performance problem. For a focused development measurement, React's `<Profiler>` reports render durations for a subtree; keep this instrumentation out of ordinary production builds unless you deliberately ship a profiling-enabled build.

```tsx
import { Profiler } from "react";

function reportRender(
  id: string,
  phase: "mount" | "update" | "nested-update",
  actualDuration: number,
  baseDuration: number,
) {
  if (import.meta.env.DEV) {
    console.debug(`${id} (${phase})`, { actualDuration, baseDuration });
  }
}

<Profiler id="Catalog" onRender={reportRender}>
  <ProductGrid products={products} />
</Profiler>
```

`actualDuration` measures work for the committed update; `baseDuration` estimates the subtree cost without memoization. Use the Profiler panel and browser Performance tools to understand the cause, then remove noisy diagnostics after the investigation.
## 3. React Compiler and manual memoization

The verified stack includes `babel-plugin-react-compiler` 1.0.0 for a measured/controlled introduction. Compiler diagnostics can reveal unsupported patterns and purity issues. Compilation can automatically memoize work; it does not fix incorrect state ownership, mutation, expensive network waterfalls, huge DOM trees, or an unstable API contract.

When enabling it, use the official installation path for the actual Vite/React plugin versions; review the generated transform and compare production behavior. Keep Hooks lint/compiler diagnostics enabled. Do not remove correct dependencies or suppress a diagnostic to force compilation.

Manual `memo`, `useMemo`, and `useCallback` remain tools for a measured case:

- `memo` can skip some child renders when props are referentially stable.
- `useMemo` can cache an expensive pure calculation or stable value where identity matters.
- `useCallback` can stabilize a callback for a memoized consumer or Hook dependency.

Each also adds mental/maintenance cost. A changed object/function dependency invalidates the cache. Compiler-enabled projects may need less manual memoization; verify its output and profiler evidence.

## 4. Network, image, and bundle performance

- Keep route-level code split where routes are independent and large.
- Avoid request waterfalls: fetch independent data in parallel or prefetch only when a likely journey supports the cost.
- Use appropriately sized responsive images, dimensions/aspect ratios to reduce layout shift, and lazy-load below-the-fold media.
- Keep server pagination for large user/product datasets.
- Use Query caching deliberately; stale-while-revalidate can improve UX, but don't show data after permissions/session change.
- Set budgets for JavaScript chunk size, image bytes, and route-critical requests based on real device profiles.

## 5. Virtualize only when measured

Virtualization reduces DOM nodes for very large lists, but complicates item focus, screen-reader traversal, dynamic row heights, selection, scroll restoration, and keyboard navigation. Use TanStack Virtual only when profiling shows DOM volume is a bottleneck and the design has an accessible navigation strategy. Pagination is often simpler for admin tables.

## Debugging lab

- `useMemo` added but interaction is slower: profile work may be network/layout, not the calculation.
- LCP poor with a fast React render: inspect image size, font loading, server response, and critical network waterfall.
- INP high on a table: inspect synchronous sorting/filters, long tasks, and row count; use server processing or deferred render where appropriate.
- CLS spikes after images load: reserve dimensions/aspect ratio and inspect late-inserted content.
- Compiler diagnostic fires on impure render: fix mutation/side effects; don't suppress the warning.

## Exercises

1. Capture a baseline for catalogue filtering with 1,000 representative rows.
2. Use browser performance tools to classify the slow path before optimization.
3. Split a heavy admin analytics route and compare transferred JavaScript.
4. Compare manual memoization before/after React Compiler with the Profiler.
5. Decide whether the admin table needs pagination or virtualization; justify by measurements and accessibility cost.
6. Write a short performance report with device/network profile, baseline, change, result, and remaining risk.

## Summary and production tips

**Summary:** performance is a measured user outcome. Start with the bottleneck; optimize the route/network/DOM/render layer responsible; measure again. React Compiler reduces some manual memo work but does not replace system-level profiling.

```text
journey + budget → measure → locate bottleneck → smallest justified change → remeasure
```

- Optimize the experience, not a synthetic component count.
- Use real images/data and throttled profiles.
- Keep optimizations reversible and documented.
- Don't optimize every component or virtualize every list.
- Track field metrics after deployment as well as lab measurements.

## Official documentation

- [React Developer Tools](https://react.dev/learn/react-developer-tools)
- [React Profiler](https://react.dev/reference/react/Profiler)
- [React Compiler](https://react.dev/learn/react-compiler)
- [React Compiler installation](https://react.dev/learn/react-compiler/installation)
- [React Hooks ESLint plugin/compiler diagnostics](https://react.dev/reference/eslint-plugin-react-hooks)
- [Vite build guide](https://vite.dev/guide/build)
- [Web Vitals](https://web.dev/articles/vitals)
- [Image performance](https://web.dev/learn/images/)
- [TanStack Virtual](https://tanstack.com/virtual/latest/docs/framework/react/introduction)

## Readiness criteria

You can name the user journey and metric you are optimizing, capture and interpret a profile, distinguish React work from network/layout work, report before/after measurements, and justify not using memoization or virtualization when evidence does not support it.
