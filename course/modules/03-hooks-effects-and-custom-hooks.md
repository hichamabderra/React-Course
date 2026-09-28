# Module 3 — Hooks, Effects, Refs, Dependencies, and Custom Hooks

> **Build goal:** add a debounced catalogue search that synchronizes with a timer, then add one real external subscription and clean it up correctly. Keep filtering derived during render. Do not add a data-fetching Effect; Module 10 introduces TanStack Query.

## 1. Why Hooks exist

Hooks let function components use React-managed state and connect to capabilities such as context, refs, and Effects. They are not lifecycle callbacks to memorize. The Rules of Hooks provide a stable order so React can associate each Hook call with its stored state.

```tsx
function ProductCount({ products }: { products: readonly Product[] }) {
  const count = products.length;
  return <p>{count} products</p>;
}
```

Hooks must be called at the top level of a component or custom Hook, not inside a conditional, loop, nested function, or event handler. Extract a component or custom Hook when different branches genuinely need different stateful behavior.

## 2. An Effect synchronizes with an external system

An Effect runs after a committed render. It synchronizes React with an external system: a browser event subscription, timer, imperative widget, WebSocket, or other system whose lifecycle is outside React. The setup function may return cleanup; cleanup runs before the Effect re-synchronizes and when the component leaves the tree.

```tsx
useEffect(() => {
  const id = window.setInterval(() => setNow(Date.now()), 1_000);
  return () => window.clearInterval(id);
}, []);
```

The dependency list is a declaration of the reactive values used by the Effect, not a schedule you choose to suppress inconvenient reruns. When dependencies change, React cleans up the old synchronization and establishes the new one.

### Derivation is not synchronization

```tsx
// Avoid: derived state plus an Effect creates two representations.
const [visible, setVisible] = useState<Product[]>([]);
useEffect(() => setVisible(selectProducts(products, search)), [products, search]);

// Prefer: calculate the view from its sources.
const visible = selectProducts(products, search);
```

`useMemo` can cache a pure calculation after measurement; it does not make an Effect necessary. A user event that changes a value belongs in the handler that processes the event, not in an Effect that watches the resulting state and tries to infer what the user did.

## 3. Debounce is external timing

A debounce timer is an external resource, so an Effect can own the timer lifecycle. The filtered data remains derived; only the delayed search value is state.

```tsx
function useDebouncedValue<T>(value: T, delayMs: number): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const timeoutId = window.setTimeout(() => setDebounced(value), delayMs);
    return () => window.clearTimeout(timeoutId);
  }, [value, delayMs]);

  return debounced;
}
```

The cleanup cancels an older timer when a newer keystroke arrives. This is a good custom Hook because it captures one reusable external-system lifecycle; it does not hide the business rules of the catalogue.

Debouncing and `useDeferredValue` solve different problems. Debounce delays when work begins, often to reduce request count. Deferred rendering lets React prioritize urgent updates while expensive UI catches up. A debounce can feel slow; a deferred value does not guarantee fewer network requests.

## 4. Refs are mutable handles, not render state

`useRef` stores a value that persists between renders without causing a render when it changes. Use it for imperative handles (DOM node), timer IDs, or a value needed across event callbacks but not displayed. If changing a value should change the UI, use state instead.

```tsx
const inputRef = useRef<HTMLInputElement>(null);

function focusSearch() {
  inputRef.current?.focus();
}

return <input ref={inputRef} type="search" />;
```

Do not read or write refs during render to drive visible output. The render should remain pure.

## 5. Dependencies, closures, and stale values

A callback or Effect closes over values from the render that created it. The lint rule `react-hooks/exhaustive-deps` catches missing synchronization inputs; silencing it can freeze old values into a listener or timer.

**Deliberate bug:**

```tsx
useEffect(() => {
  const id = window.setInterval(() => {
    setCount(count + 1);
  }, 1_000);
  return () => window.clearInterval(id);
}, []); // count is captured from the first render
```

**Fix for this state transition:**

```tsx
setCount((current) => current + 1);
```

Another valid solution may be to re-establish an external subscription when a reactive value changes. Choose based on the external system's semantics, not merely to make lint quiet.

React 19.3's `useEffectEvent` is for event logic that is called from an Effect and must read the latest committed values without re-subscribing for those values. It is **not** a general escape hatch from dependencies, and its returned function must not be passed to children or called during render.

```tsx
const notifyLatestTheme = useEffectEvent((message: string) => {
  showNotification(message, theme);
});

useEffect(() => {
  const connection = connect(roomId);
  connection.on("message", notifyLatestTheme);
  return () => connection.disconnect();
}, [roomId]);
```

Use this only if `theme` affects notification presentation but should not reconnect the room. If changing the value should change the connection itself, it belongs in the dependency list.

## 6. Effects that fetch: understand the risks, then prefer Query

A one-off fetch Effect can produce races, duplicate requests, stale responses, missing loading/error states, and cache duplication. If teaching the primitive, cancellation and response ordering must be explicit:

```tsx
useEffect(() => {
  const controller = new AbortController();
  let current = true;

  async function load() {
    try {
      const response = await fetch(`/api/products?q=${encodeURIComponent(search)}`, {
        signal: controller.signal,
        credentials: "include",
      });
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      const payload: unknown = await response.json();
      if (current) setResult(parseCatalog(payload));
    } catch (error) {
      if (current && !controller.signal.aborted) setError(normalizeError(error));
    }
  }

  void load();
  return () => {
    current = false;
    controller.abort();
  };
}, [search]);
```

This example is a debugging bridge, not the capstone's final data architecture. Runtime parsing and request cancellation are still required. Module 10 replaces this ad hoc cache/state lifecycle with Query's shared server-data lifecycle.

## Debugging lab

- **Effect loop:** an Effect sets state every time it runs. Ask whether the new state is derived, whether an object/function dependency is recreated every render, and whether synchronization is actually needed.
- **Stale timer:** a timer callback reads old state. Use a functional updater or intentionally re-subscribe.
- **Race:** type `lamp`, then `lamp shade`; the older response arrives last. Abort the older work and ensure only current results are displayed.
- **Strict Mode duplicate setup:** in development React may perform an extra setup/cleanup cycle. A correct Effect tolerates setup → cleanup → setup. Do not disable Strict Mode to hide a missing cleanup.

## Exercises

1. Implement `useDebouncedValue`; verify rapid typing leaves one trailing update.
2. Add and remove a `visibilitychange` listener. Confirm cleanup on unmount.
3. Reproduce and fix the interval stale-closure bug.
4. Add a deliberately incorrect dependency omission; use the lint diagnostic to find the synchronization bug.
5. Explain why `useEffect(() => setTotal(items.reduce(...)), [items])` is worse than deriving `total` during render.

## Summary, mental model, and production tips

**Summary:** Hooks follow stable call order. State is for render-affecting facts; refs are mutable non-render handles; Effects synchronize with external systems and clean up. Dependencies describe reactive inputs. Derived values belong in render.

```text
render calculation → no side effects
user event → perform event-caused work
Effect setup/cleanup → synchronize external lifecycle
```

- Prefer native event handlers to Effects that infer an event afterward.
- Do not add `useMemo` or `useCallback` just to satisfy a vague performance instinct.
- Keep custom Hooks narrow and name them after a behavior, not a data bucket.
- Treat cancellation, unsubscribe, and cleanup as correctness, not optional polish.
- Never silence Hook lint before explaining why every reactive value is safe.

## Official documentation

- [Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks)
- [Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
- [Lifecycle of Reactive Effects](https://react.dev/learn/lifecycle-of-reactive-effects)
- [Removing Effect Dependencies](https://react.dev/learn/removing-effect-dependencies)
- [useEffect](https://react.dev/reference/react/useEffect), [useRef](https://react.dev/reference/react/useRef), [useMemo](https://react.dev/reference/react/useMemo), [useCallback](https://react.dev/reference/react/useCallback)
- [useEffectEvent](https://react.dev/reference/react/useEffectEvent)
- [React Hooks ESLint plugin](https://react.dev/reference/eslint-plugin-react-hooks)
- [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)

## Readiness criteria

You can defend each Effect by naming the external system it synchronizes with; explain each dependency; show cleanup; tell a state value from a ref; diagnose a stale closure; and remove derived-state Effects without losing behavior.
