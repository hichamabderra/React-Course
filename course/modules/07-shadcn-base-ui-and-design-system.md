# Module 7 — shadcn/ui, Base UI, and an Accessible Design System

> **Build goal:** add a small source-owned design-system foundation—Button, TextField, StatusBadge, Dialog, and PageHeader—then reuse it in catalogue/account/admin contexts without hiding each feature's meaning.

## 1. What shadcn/ui is—and is not

shadcn/ui is a CLI/registry workflow that adds component source code to your project. It is not a closed library whose internals you cannot inspect. The generated files become application code: review them, update them, test them, and own the maintenance cost.

As of the verified September 2026 stack, new shadcn projects use **Base UI** by default. Radix remains supported, and an existing Radix application does not need a migration just because the default changed. Always choose the component primitive base explicitly for an existing project and read the matching docs; APIs are not guaranteed to be drop-in compatible.

## 2. Start with native HTML, add a primitive when behavior is complex

A button, label, input, and table already have browser behavior. A complex dialog/menu/combobox additionally needs focus management, keyboard interactions, dismissal, and relationships. That is where a tested primitive helps.

Do not recreate these by adding `role="dialog"` to a div. ARIA can describe semantics but does not implement focus trapping, Escape behavior, modal inertness, or restoring focus.

A simple shared Button can add visual variants without changing native semantics:

```tsx
import { Button } from "@/components/ui/button";

<Button type="button" variant="outline" onClick={openProductEditor}>
  Edit product
</Button>
```

The exact props are the generated component's public API; inspect the source before assuming a variant or polymorphic behavior exists. Prefer a real `<button>` for actions and a router `<Link>` for navigation.

## 3. Build tokens and variants with intent

A shared TextField should accept a label, optional description, error message, and native input props. It should connect accessible names and error descriptions rather than merely drawing a red border.

```tsx
import type { ComponentProps } from "react";

interface TextFieldProps extends Omit<ComponentProps<"input">, "id"> {
  id: string;
  label: string;
  description?: string;
  error?: string;
}

function TextField({ id, label, description, error, ...inputProps }: TextFieldProps) {
  const descriptionId = description ? `${id}-description` : undefined;
  const errorId = error ? `${id}-error` : undefined;

  return (
    <div className="field">
      <label htmlFor={id}>{label}</label>
      {description && <p id={descriptionId}>{description}</p>}
      <input
        {...inputProps}
        id={id}
        aria-invalid={Boolean(error)}
        aria-describedby={[descriptionId, errorId].filter(Boolean).join(" ") || undefined}
      />
      {error && <p id={errorId}>{error}</p>}
    </div>
  );
}
```

This wrapper is useful only if it preserves native props and labels consistently. Avoid building a wrapper that hides input `name`, `type`, `autoComplete`, `required`, or `disabled` behavior.

## 4. Dialog anatomy and focus expectations

For a product edit dialog, verify these behaviors from the selected Base UI/shadcn docs and keyboard testing:

1. The trigger has a clear accessible name.
2. Opening places focus inside the dialog.
3. The dialog has a labelled title and useful description.
4. Tab/Shift+Tab stay in the modal's focus scope if it is modal.
5. Escape/cancel closes it when permitted.
6. Closing restores focus to the trigger when it still exists.
7. A successful save reports status and closes only when that is the intended workflow.
8. A validation failure keeps the dialog open and identifies errors.

Use the official generated Dialog component and its current API. Component APIs can change across primitive bases; do not mix Radix examples into a Base UI component.

## 5. Source ownership and update strategy

- Keep generated primitives in a `shared/ui` folder only when multiple features actually use them.
- Keep domain text/permissions/actions in the feature that owns the workflow.
- Add only the components required for the current slice; do not generate the full catalog preemptively.
- Review generated code and licenses; copied source remains your responsibility.
- Avoid customizing generated internals before reading the upstream source; smaller changes are easier to rebase.
- Use design tokens and semantic variants (`primary`, `danger`, `quiet`) instead of one-off colors.

## Debugging lab

- A dialog opens but keyboard focus stays behind it: inspect the primitive and whether the correct modal mode was configured; do not patch with `document.querySelector`.
- A field's error is visible but not announced: verify `aria-describedby`, `aria-invalid`, focus behavior, and whether the error text enters the accessible tree.
- A copied component accepts an `asChild` prop in an old tutorial but your current Base UI version does not: use the current generated API, not an old Radix tutorial.
- A shared Button has many feature-specific boolean props: move behavior to composition or feature-owned wrappers.

## Exercises

1. Add Button variants for primary, destructive, outline, and quiet. Check contrast and focus in light/dark themes.
2. Build the TextField wrapper above and test label/error relationships with Testing Library in Module 17.
3. Add a product edit Dialog using the generated Base UI component; perform the seven focus checks manually.
4. Add a loading state to a Button without changing its accessible name unexpectedly.
5. Inspect a generated source file and record one upstream primitive behavior your team must preserve.

## Summary and production tips

**Summary:** shadcn gives source ownership; Base UI supplies tested interaction primitives; native elements remain the default for simple controls. Shared components should standardize behavior without absorbing feature logic.

```text
native semantic HTML → primitive for complex interaction → owned source → design-system variant → feature composition
```

- Test behavior, not only screenshots.
- Use the current base's official API; Radix and Base UI are alternatives, not snippets to mix.
- Accessibility is a behavior contract, not a role attribute or CSS class.
- Do not add a design-system layer until a real second consumer exists.

## Official documentation

- [shadcn Vite installation](https://ui.shadcn.com/docs/installation/vite)
- [shadcn CLI](https://ui.shadcn.com/docs/cli)
- [shadcn theming](https://ui.shadcn.com/docs/theming)
- [shadcn component catalog](https://ui.shadcn.com/docs/components)
- [July 2026 Base UI default announcement](https://ui.shadcn.com/docs/changelog/2026-07-base-ui-default)
- [Base UI React](https://base-ui.com/react) and [components](https://base-ui.com/react/components)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [React input component](https://react.dev/reference/react-dom/components/input)

## Readiness criteria

You can explain why shadcn source is application code; identify which element should be native versus a primitive; implement a labelled/error-aware field; test dialog focus behavior; and choose Base UI or Radix intentionally without treating generated code as a black box.
