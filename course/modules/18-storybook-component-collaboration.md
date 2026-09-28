# Module 18 — Storybook, Component Collaboration, and Isolated States

> **Build goal:** document a small set of reusable Northstar components—Button, TextField, Dialog, ProductCard, StatusBadge, and a table row—with representative states and interaction examples. Storybook is optional; keep it only if it helps the team collaborate or review components.

## 1. What Storybook adds

Storybook 10.6.0 runs UI components outside the full app. This makes states visible: default, loading, disabled, invalid, empty, long text, dark theme, reduced motion, and permission-limited. It supports documentation, interaction checks, accessibility add-ons, and visual review.

Storybook is not a replacement for integration tests or the app itself. A component that works in a story can still fail with router/provider/query/API integration. Avoid maintaining a parallel component system that only exists in Storybook.

## 2. Stories are executable examples

Storybook's Component Story Format lets a story describe a named input/state for a component.

```tsx
import type { Meta, StoryObj } from "@storybook/react-vite";
import { ProductCard } from "./ProductCard";
import type { Product } from "./product";

const product = {
  id: "prod-1001",
  name: "Orbit Studio Headphones",
  category: "audio",
  description: "Clear sound for focused work.",
  price: { amountMinor: 12900, currency: "USD" },
  stock: "in-stock",
} satisfies Product;

const meta = {
  component: ProductCard,
  args: { product },
} satisfies Meta<typeof ProductCard>;

export default meta;
type Story = StoryObj<typeof meta>;

export const InStock: Story = {};
export const SoldOut: Story = {
  args: { product: { ...product, stock: "sold-out" } },
};
```

This snippet assumes `ProductCard` renders a product without fetching it. Keep the story fixture typed, stable, and close to the domain shape. Avoid inventing a second set of story-only props.

## 3. Stories reveal state coverage

For interactive components, add `play` behavior for important interactions using the current Storybook test APIs. Keep the same accessible role/label expectations used in Testing Library. For global providers (theme, router, Query), add a focused decorator only where the component actually needs that context.

Useful story matrix:

| Component | States worth documenting |
|---|---|
| Button | Primary/destructive/quiet, disabled, pending, icon-only accessible name |
| TextField | Empty, help text, invalid, server error, disabled, long label |
| Dialog | Open, validation error, pending submit, confirmed close, keyboard/focus behavior |
| ProductCard | Long name, low stock, sold out, missing optional image |
| Admin row | Allowed action, forbidden UX, pending mutation, conflict |

Do not encode authorization into story decorators as if the backend would enforce it; a story demonstrates the UI state only.

## 4. Design review and token/theme coverage

Stories can be viewed at responsive widths and in light/dark theme. They help designers/engineers discuss visual variants before a feature is routed. They do not prove contrast, keyboard operation, screen-reader announcement, or production data correctness. Use automated accessibility checks plus manual review.

## 5. Cost and when not to adopt

Storybook adds configuration, dependency updates, build/CI time, and a requirement to keep stories maintained. It is justified when shared components have several meaningful states or design review benefits. For a small private page with no shared component collaboration need, focused tests and the app may be enough.

A story becomes stale when it duplicates actual component state instead of expressing it through props/context. Remove dead stories and update fixtures when domain contracts change.

## Debugging lab

- Story fails because the component expects QueryClient: decide whether the component boundary is too data-coupled; add a provider only if that is the real app contract.
- Story builds but CSS differs from the app: verify global styles, Tailwind content detection, and theme decorators.
- `play` test passes while keyboard interaction is broken: check that the story actually performs keyboard behavior rather than clicking only.
- A story manually adds `role="dialog"` around a component: test the real primitive and its focus semantics instead of wrapping to make the snapshot look right.

## Exercises

1. Add light and dark stories for Button and TextField.
2. Add a dialog story with initial, invalid, and pending states.
3. Add a play interaction that submits an invalid field and checks the visible accessible error.
4. Run accessibility checks in Storybook and record what still needs manual review.
5. Ask a peer to use the stories to identify a missing state; decide whether it merits a new story or a test instead.

## Summary and production tips

**Summary:** Storybook makes component states inspectable and shareable. It pays off when collaboration and state coverage justify maintaining it; it complements, not replaces, the app and test suite.

```text
component contract → story inputs/states → interaction review → app integration test
```

- Keep story fixtures aligned with production types.
- Show error, empty, pending, disabled, and responsive states.
- Avoid story-only component APIs and provider boilerplate.
- Use interaction tests for behavior and manual review for meaning/context.
- Revisit Storybook's current framework/test integrations when upgrading versions.

## Official documentation

- [Storybook React + Vite framework](https://storybook.js.org/docs/get-started/frameworks/react-vite)
- [Writing stories](https://storybook.js.org/docs/writing-stories)
- [Storybook documentation](https://storybook.js.org/docs/writing-docs)
- [Interaction testing](https://storybook.js.org/docs/writing-tests/interaction-testing)
- [Accessibility testing](https://storybook.js.org/docs/writing-tests/accessibility-testing)
- [Vitest addon](https://storybook.js.org/docs/writing-tests/integrations/vitest-addon)
- [Storybook 10 migration guide](https://storybook.js.org/docs/releases/migration-guide)

## Readiness criteria

You can create stories from real component props, cover representative state variants, write an interaction story using accessible queries, use decorators only for real providers, and explain when Storybook's maintenance cost is not justified.
