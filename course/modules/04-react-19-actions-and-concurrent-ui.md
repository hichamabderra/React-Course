# Module 4 — React 19.3 Actions, Transitions, Suspense, and Compiler

> **Build goal:** add a responsive optimistic favorite/cart-quantity interaction, a pending form action, and one lazy route boundary. Do not imply a payment or order succeeded until the backend confirms it. This is a Vite SPA; React Server Components are studied for architecture decisions, not hand-built here.

## 1. Scheduling is part of user experience

A browser event handler should respond promptly. Some state changes should render urgently (typing into an input); other rendering can be deferred (filtering a huge list). React transitions let us mark updates as non-urgent. A transition does not move CPU work to another thread and does not make an API secure.

```tsx
import { useDeferredValue, useState } from "react";

const [query, setQuery] = useState("");
const deferredQuery = useDeferredValue(query);
const results = selectCatalogProducts(products, deferredQuery);

<input value={query} onChange={(event) => setQuery(event.currentTarget.value)} />
{query !== deferredQuery && <p aria-live="polite">Updating results…</p>}
```

The input updates immediately; result rendering may lag. The pending indicator communicates that distinction. For a small catalogue, this optimization is unnecessary; keep it as a measured option.

## 2. Transitions and Actions

`startTransition` marks synchronous updates as non-urgent. `useTransition` provides an `isPending` flag. An Action is a function passed to certain React APIs or called inside a transition; async Actions can represent pending work and optimistic UI. An Action is not the same as a backend transaction or authorization check.

```tsx
import { useTransition } from "react";

const [isPending, startTransition] = useTransition();

function chooseCategory(next: ProductCategory | "all") {
  startTransition(() => {
    setCategory(next);
  });
}
```

Do not mark every state update as a transition. Input echo, focus, and immediate feedback should remain urgent.

React 19.3 also allows functions as form `action` props. For this capstone, React Hook Form owns richer validation workflows in Module 13; native React Actions are taught as an alternative for simpler form submissions and progressive enhancement where a server framework supports them.

```tsx
import { useActionState } from "react";

interface SaveState {
  message: string;
}

async function saveDisplayName(
  _previous: SaveState,
  formData: FormData,
): Promise<SaveState> {
  const displayName = String(formData.get("displayName") ?? "").trim();
  if (!displayName) return { message: "Enter a display name." };
  await updateProfile({ displayName });
  return { message: "Profile saved." };
}

function DisplayNameForm() {
  const [state, formAction, isPending] = useActionState(saveDisplayName, { message: "" });
  return (
    <form action={formAction}>
      <label>
        Display name
        <input name="displayName" />
      </label>
      <button disabled={isPending}>{isPending ? "Saving…" : "Save"}</button>
      <p role="status">{state.message}</p>
    </form>
  );
}
```

A form action is invoked by React. The example's `updateProfile` is an application API function; its server validation remains authoritative. A production version must represent field errors, preserve safe input, and normalize network failures.

## 3. Optimistic UI is temporary, not truth

`useOptimistic` renders an intended state while an Action is pending. The canonical value still comes from the successful server response or refreshed Query cache. For one small interaction, a UI-only optimistic value is often easier than rewriting every matching cache entry.

```tsx
import { useOptimistic, useState, useTransition } from "react";

function QuantityControl({ quantity, onSave }: {
  quantity: number;
  onSave: (next: number) => Promise<void>;
}) {
  const [optimisticQuantity, setOptimisticQuantity] = useOptimistic(quantity);
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | null>(null);

  function update(next: number) {
    setError(null);
    startTransition(async () => {
      setOptimisticQuantity(next);
      try {
        await onSave(next);
      } catch {
        setError("We could not update the cart. The saved quantity is unchanged.");
      }
    });
  }

  return (
    <div>
      <button type="button" disabled={isPending} onClick={() => update(Math.max(1, optimisticQuantity - 1))}>
        Decrease quantity
      </button>
      <output aria-live="polite">{optimisticQuantity}</output>
      <button type="button" disabled={isPending} onClick={() => update(optimisticQuantity + 1)}>
        Increase quantity
      </button>
      {error && <p role="alert">{error}</p>}
    </div>
  );
}
```

This is a teaching sketch; the final Query-backed version belongs in Module 10. On success, `onSave` must update the canonical Query cache/parent prop before the Action ends, or the optimistic quantity will correctly fall back to the old base value. Avoid optimistic updates when the outcome is risky, difficult to reverse, or financially consequential. Never mark checkout/payment complete before authoritative confirmation. Concurrent operations need a deliberate merge/rollback policy.

## 4. Suspense, lazy loading, and errors

`lazy` defers loading a component module until it is rendered. `Suspense` owns the pending fallback. An error boundary handles rendering errors; it is not a catch-all for every event handler or arbitrary async request.

```tsx
import { lazy, Suspense } from "react";

const AnalyticsPage = lazy(() => import("./features/admin/AnalyticsPage"));

<Suspense fallback={<PageSkeleton label="Loading analytics" />}>
  <AnalyticsPage />
</Suspense>
```

Place a boundary at a useful user-experience boundary. A single full-screen spinner for the whole app can hide already usable navigation. Error boundaries should offer recovery (retry/reload/navigation) and log a correlation identifier without exposing internals.

## 5. Newer React capabilities: know the problem before using them

- **`useActionState` / form Actions:** tie the result and pending state of a form action to its UI.
- **`useOptimistic`:** show temporary intent during an Action and return to canonical state afterward.
- **`useDeferredValue`:** let expensive rendering lag behind an urgent value; it is not debounce.
- **`Activity`:** preserve hidden subtree state/effects semantics deliberately; it is not a default replacement for conditional rendering.
- **`ViewTransition`:** coordinate browser view-transition animations with React transitions/Suspense. Check browser support, route integration, naming, and reduced motion.
- **Fragment refs:** allow certain DOM interactions across a group without adding a wrapper; prefer ordinary element refs when possible.
- **React Compiler:** compiler-guided memoization can reduce manual memo work. It cannot fix wrong ownership, mutation, effects, or network waterfalls.
- **RSC/Server Functions:** valuable in framework/server-rendered architectures. The capstone's Vite SPA deliberately uses a separate REST backend and does not configure an RSC bundler.

## Debugging lab

1. Use `useDeferredValue` on a short static list; notice that it adds complexity without a measurable gain.
2. Make an optimistic update fail. Verify the canonical value returns, a clear message appears, and a retry remains possible.
3. Put an invalid route chunk behind `lazy`; confirm the nearest error boundary handles the import failure.
4. Start two updates quickly; decide whether to serialize them, disable duplicate submissions, or use server versions/idempotency.

## Exercises

- Add `useTransition` to category navigation and visually announce pending results without disabling keyboard input.
- Build a `useActionState` form that returns field errors as structured state.
- Add an optimistic favorite toggle that rolls back on a simulated `403` or network failure.
- Compare a route-level `Suspense` fallback with a page-wide fallback using the browser network throttle.
- Write a short decision note: which newer React APIs will Northstar actually use, and which add no value today?

## Production tips, summary, and mental model

- Use transitions to protect urgent interactions, not as a universal wrapper.
- Give pending work a visible state and prevent accidental duplicate mutation.
- Treat optimistic UI as a temporary projection; reconcile with the server.
- Keep Suspense/error boundaries close enough to preserve unaffected UI.
- Follow the documented framework integration for RSC; do not create private bundler infrastructure for a course SPA.

```text
urgent input → immediate state
non-urgent render → transition/deferred value
mutation in flight → optimistic projection + pending signal
server response → canonical state, success, or rollback/error
```

**Summary:** React 19.3 offers Actions and better primitives for pending/optimistic UI. Their value depends on the use case; they do not replace Query, API validation, or business/security rules.

## Official documentation

- [React 19.3 release notes](https://react.dev/blog/2026/09/09/react-19-3)
- [useTransition](https://react.dev/reference/react/useTransition) and [startTransition](https://react.dev/reference/react/startTransition)
- [useActionState](https://react.dev/reference/react/useActionState)
- [useOptimistic](https://react.dev/reference/react/useOptimistic)
- [useDeferredValue](https://react.dev/reference/react/useDeferredValue)
- [React `<form>`](https://react.dev/reference/react-dom/components/form) and [useFormStatus](https://react.dev/reference/react-dom/hooks/useFormStatus)
- [Suspense](https://react.dev/reference/react/Suspense), [lazy](https://react.dev/reference/react/lazy), and [Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [Activity](https://react.dev/reference/react/Activity), [ViewTransition](https://react.dev/reference/react/ViewTransition), and [Fragment](https://react.dev/reference/react/Fragment)
- [React Compiler](https://react.dev/learn/react-compiler)
- [React Server Components](https://react.dev/reference/rsc/server-components)

## Readiness criteria

You can explain what a transition changes (priority, not trust or CPU threads), where pending state belongs, how optimistic state converges or rolls back, why Suspense needs an error boundary partner, and why this Vite capstone does not hand-roll Server Components.
