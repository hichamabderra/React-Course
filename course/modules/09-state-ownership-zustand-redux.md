# Module 9 — State Ownership, Zustand, and Redux Toolkit

> **Build goal:** add a guest-cart draft that can follow a shopper across routes, then hand it to the server for validation/merge after login. Use one shared client store for that narrow case; leave filters in the URL, server cart in Query, forms in RHF, and dialog state local.

## 1. Classify before choosing a store

Ask what the value represents and who is authoritative:

| Example | Owner | Reason |
|---|---|---|
| Quick-view open/closed | Component | Ephemeral UI for one view. |
| Profile field values/errors | Form library | Form lifecycle and field validation. |
| Catalogue query/sort/page | URL | Shareable/navigation state. |
| Product records/orders/server cart | TanStack Query + backend | Remote data, freshness, permissions, invalidation. |
| Guest cart draft | Small Zustand store | Cross-route client-only state until server merge. |

“Global” is not a reason to use global state. A value used by a header and a page may simply belong to a route layout or a Query cache.

## 2. Zustand for a narrow client-only store

Zustand 5.0.15 uses a small store and selector-based subscriptions. Keep the guest draft minimal: product identifiers and quantity, no address, payment data, prices, role, or credential.

```ts
import { create } from "zustand";

type GuestCartLine = { productId: string; quantity: number };
type GuestCartState = {
  lines: Record<string, number>;
  setQuantity: (productId: string, quantity: number) => void;
  remove: (productId: string) => void;
  clear: () => void;
};

export const useGuestCart = create<GuestCartState>((set) => ({
  lines: {},
  setQuantity: (productId, quantity) =>
    set((state) => ({ lines: { ...state.lines, [productId]: quantity } })),
  remove: (productId) =>
    set((state) => {
      const lines = { ...state.lines };
      delete lines[productId];
      return { lines };
    }),
  clear: () => set({ lines: {} }),
}));
```

The update returns a new object; it does not mutate `state.lines`. Components select the smallest needed value:

```tsx
const quantity = useGuestCart((state) => state.lines[productId] ?? 0);
const setQuantity = useGuestCart((state) => state.setQuantity);
```

Zustand v5 compares selector output with `Object.is`. A selector that returns a newly allocated object on every render can trigger avoidable updates. Select individual fields or use the documented `useShallow` helper when a shallow object selection is useful.

## 3. Persistence is an explicit product/security choice

A guest cart can be useful after refresh. If we persist it, persist only the serializable draft fields and validate on rehydration. Local storage is browser-readable and user-editable, so it is not a trusted database. The server revalidates each product and quantity during merge.

```ts
// Conceptual allowlist when adding Zustand persist:
partialize: (state) => ({ lines: state.lines })
```

Do not persist session/access/refresh credentials in this store. Never persist a trusted price or assume client quantity is within server limits. Consider privacy/retention expectations and the risk of stale product identifiers.

## 4. Handoff to server state

On successful login, send the guest draft to the merge endpoint, then replace the UI with the returned canonical cart. Clear the guest draft only after the server confirms the merge or according to a documented conflict strategy.

```tsx
const lines = useGuestCart((state) => state.lines);
const mergeCart = useMutation({
  mutationFn: () => api.post("/cart/merge", { lines }),
  onSuccess: (serverCart) => {
    queryClient.setQueryData(["cart"], serverCart);
    useGuestCart.getState().clear();
  },
});
```

This abbreviated example illustrates ownership; in actual code use the app's typed endpoint/query-key factory and consider partial success/conflict responses. Once authenticated, the server owns the cart. Do not keep a second authoritative cart copy in Zustand.

## 5. Redux Toolkit comparison

Redux Toolkit is a strong choice when an organization benefits from explicit reducer/event conventions, middleware, action history, standardized tooling, or RTK Query as its server-state solution. Zustand is selected here for one narrow shared client-only state case. They are alternatives, not two global stores the capstone needs simultaneously.

If a project adopts RTK Query, evaluate whether it should own remote data instead of TanStack Query. Two server caches create duplicate invalidation and freshness concepts unless a concrete boundary justifies them. Context is often sufficient for low-frequency dependencies; local state is often sufficient for UI.

## Debugging lab

- Selector returns `{ quantity, setQuantity }` inline and the component re-renders continuously: check fresh object identity and use narrow selectors/`useShallow`.
- Guest cart shows a product that is now deleted: the draft is not authoritative; merge should return a conflict and the UI must reconcile.
- User signs in on one route, but header count remains stale: check which owner supplies the count and whether the server cart cache updates.
- State persists a sensitive field: remove it from persisted serialization; clear old storage and treat past values as exposed.

## Exercises

1. Add a quantity validator that keeps local drafts between 1 and 99, while the server still validates its own maximum.
2. Add `remove` and derive a cart count without storing a redundant `itemCount` field.
3. Write a merge state diagram for success, partial conflict, network failure, and logout.
4. Compare a small Zustand store with a React context/reducer implementation; identify the actual complexity difference.
5. Explain what should happen to the guest cart after logout from an authenticated session.

## Summary, mental model, and production tips

**Summary:** state belongs to the owner that can update it consistently. Zustand is reserved for a cross-route client draft; Query owns authenticated server data; URL/form/local state stay in their respective boundaries.

```text
local UI → component
shareable view → URL
remote entity → Query + server
cross-route client draft → small store
```

- Select narrowly; avoid subscribing a whole layout to every cart field.
- Persist only what has a product reason and a safe rehydration policy.
- Clear or reconcile client-only state at authentication transitions.
- Do not store money as authority, roles as authorization, or credentials in a client store.
- Add Redux Toolkit only when its conventions/tooling solve a real team-scale problem.

## Official documentation

- [React choosing state structure](https://react.dev/learn/choosing-the-state-structure)
- [React sharing state](https://react.dev/learn/sharing-state-between-components)
- [Zustand introduction](https://zustand.docs.pmnd.rs/getting-started/introduction)
- [Zustand immutable state and merging](https://zustand.docs.pmnd.rs/guides/immutable-state-and-merging)
- [Zustand `useShallow`](https://zustand.docs.pmnd.rs/guides/prevent-rerenders-with-use-shallow)
- [Zustand v5 migration](https://zustand.docs.pmnd.rs/reference/migrations/migrating-to-v5)
- [Redux Toolkit getting started](https://redux-toolkit.js.org/introduction/getting-started)
- [Redux Essentials](https://redux.js.org/tutorials/essentials/part-1-overview-concepts)
- [RTK Query overview](https://redux-toolkit.js.org/rtk-query/overview)

## Readiness criteria

You can justify each state owner in the table, implement an immutable narrow store, explain selector identity, keep credentials/prices/roles out of it, and describe the guest-to-server cart handoff including failure cases.
