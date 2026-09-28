# Module 15 — Accessibility as an Engineering Constraint

> **Build goal:** audit and remediate one end-to-end customer journey and one admin workflow using keyboard-only navigation, zoom, an automated scan, and manual screen-reader review. Accessibility is part of the acceptance criteria for each feature, not a one-time pass at the end.

## 1. Model the user task, not only the element

Accessibility means a person can perceive, understand, navigate, and operate the product with different abilities, devices, and assistive technologies. A component can pass an axe scan and still have an incomprehensible flow. Start with a task: search for a product, add it, change a setting, submit a form, or manage an admin record.

WCAG 2.2 AA is the capstone product goal. The required behaviors include semantic structure, keyboard access, visible focus, meaningful names, labels/instructions, error identification, contrast, resizing/zoom, and reduced-motion support. Verify current criteria and browser/AT support during implementation.

## 2. Prefer semantics before ARIA

A real `<button>` gives focusability, keyboard activation, and semantics. A clickable `<div>` does not. A `<label>` correctly names an input. An `<a>` navigates. ARIA adds or adjusts accessible information; it does not implement interaction behavior.

```tsx
<label htmlFor="catalog-search">Search products</label>
<input id="catalog-search" type="search" />
<button type="submit">Search</button>
```

Don't add `role="button"` to a `<div>` when a button works. Don't use `aria-label` to hide a visible text name. For icon-only controls, provide a concise accessible name, and keep tooltip text supplementary rather than the only name.

## 3. Keyboard, focus, and navigation

Test a complete task with Tab, Shift+Tab, Enter, Space, Escape, and arrow keys where the component pattern requires them.

- Focus order follows visual/task order.
- Focus is visibly distinguishable in both themes.
- Dialog open moves focus into it; close restores focus to its trigger when appropriate.
- Route changes provide a sensible page title and focus/announcement behavior without stealing focus arbitrarily.
- Menus and composite widgets follow their documented keyboard pattern; do not invent one.
- Skip links and landmarks let keyboard users bypass repeated navigation.

A focus outline is part of the component's API. If a modal unmounts its trigger after an action, choose an appropriate fallback focus target.

## 4. Forms and errors

- Give controls programmatic labels and useful instructions.
- Connect errors and help text through IDs; set `aria-invalid` when invalid.
- On invalid submit, provide a summary or focus the first invalid field.
- Don't communicate validity or stock state only through red/green.
- Announce asynchronous success/error/pending without narrating every keystroke.
- Preserve entered data after recoverable errors.

`aria-live="polite"` can announce concise changing status; `role="alert"` is assertive and should be reserved for important urgent errors. Overusing live regions creates noise.

## 5. Images, tables, and dynamic content

Product photos need meaningful alternative text when they add information. Decorative art should have empty alt text or be hidden from assistive tech. Avoid repeating the product name in alt text if it is already announced adjacent to the image.

Use native table semantics for tabular data. An interactive spreadsheet-style grid requires a richer keyboard model and ARIA pattern; do not use grid roles merely for styling. For notifications/live updates, announce meaningful changes and provide a way to review a stable list.

## 6. Automated plus manual verification

Automated tools can catch missing labels, invalid ARIA, some contrast issues, and common structure problems. They cannot know whether an accessible name is meaningful, focus order matches the task, instructions make sense, a live region is too noisy, or a workflow is understandable.

Release review should combine:

1. Static lint/a11y checks.
2. Axe scan in representative states.
3. Keyboard-only completion of the task.
4. Zoom/reflow and high-contrast testing.
5. Screen-reader spot check with supported browser/AT combinations.
6. Human review of content, focus changes, and error recovery.

## Debugging lab

- Button has no name because an SVG is decorative: add visible text or an accessible name on the button.
- Form errors appear after submit but focus stays at the bottom: focus the error summary/first invalid field deliberately.
- Dialog announces as generic group: inspect title/description association and primitive configuration.
- Focus indicator disappears in dark mode: test all theme variants; don't rely on color token alone.
- Axe passes, but screen reader repeats a message on every keystroke: reduce live-region updates and announce on meaningful transitions.

## Exercises

1. Complete catalogue search and add-to-cart by keyboard.
2. Fix one form's name/description/error relationships and test invalid submit focus.
3. Review the product edit dialog's focus entry, trap, Escape, and restore behavior.
4. Test user/admin data tables at 200% zoom and a narrow viewport.
5. Run an automated axe scan, record false positives/limits, then do a manual screen-reader pass.

## Summary and production tips

**Summary:** accessible behavior arises from semantics, focus, state transitions, content, responsive layout, and assistive-technology feedback. Automated scans are one signal, not proof.

```text
task → semantic controls → keyboard/focus path → state/error announcement → manual verification
```

- Write acceptance criteria that describe accessible outcomes.
- Test with real names/content, not only placeholder text.
- Use primitives for complex interactions but verify their behavior in your app.
- Treat accessibility defects as product defects with owners and release criteria.
- Keep reduced-motion, contrast, keyboard, and zoom checks in component stories/tests.

## Official documentation

- [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/)
- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)
- [WAI-ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/)
- [WAI forms tutorial](https://www.w3.org/WAI/tutorials/forms/)
- [Accessible Name and Description Computation](https://www.w3.org/TR/accname-1.2/)
- [Keyboard interface practices](https://www.w3.org/WAI/ARIA/apg/practices/keyboard-interface/)
- [Playwright accessibility testing](https://playwright.dev/docs/accessibility-testing)
- [Storybook accessibility testing](https://storybook.js.org/docs/writing-tests/accessibility-testing)

## Readiness criteria

You can complete core tasks with keyboard alone, explain why semantics beat ARIA patches, verify names/errors/focus, identify automated testing limits, and report a specific accessibility finding with a reproducible task and expected behavior.
