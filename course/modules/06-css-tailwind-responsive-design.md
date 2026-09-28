# Module 6 — CSS Foundations, Tailwind CSS v4, Responsive Design, and Themes

> **Build goal:** establish Northstar's token system, responsive storefront/admin shell, and light/dark/system theme behavior. Keep the CSS platform visible; Tailwind is a way to author CSS, not a replacement for understanding layout or accessibility.

## 1. Begin with the browser's layout model

Use semantic HTML for meaning and CSS for presentation. Flexbox handles one-dimensional alignment; Grid handles two-dimensional track layout; normal flow remains useful for text and content. Choose based on layout, not a utility-class recipe.

```css
.catalog-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 16rem), 1fr));
  gap: 1rem;
}

.app-shell {
  display: grid;
  grid-template-columns: 16rem minmax(0, 1fr);
  min-height: 100dvh;
}

@media (max-width: 48rem) {
  .app-shell {
    grid-template-columns: minmax(0, 1fr);
  }
}
```

`minmax(0, 1fr)` prevents long content from forcing a grid column wider than the viewport. Test layouts with real content, zoom, and translated/long labels.

## 2. Design tokens represent decisions

A token gives the design system one name for a decision such as “surface,” “border,” or “focus ring.” It helps light/dark themes remain coherent and supports redesigns without changing every component.

For Tailwind 4.3.3, configuration is CSS-first through `@import "tailwindcss"` and `@theme`:

```css
@import "tailwindcss";

@theme {
  --color-ink: oklch(0.22 0.03 250);
  --color-surface: oklch(0.99 0.01 250);
  --color-brand: oklch(0.48 0.11 180);
  --breakpoint-tablet: 48rem;
  --radius-card: 1rem;
}
```

Theme variables generate utility APIs such as `bg-surface`, `text-ink`, and responsive variants. Use ordinary CSS variables for values that should not create utilities. Do not treat utility names as design documentation; keep token names semantic.

## 3. Responsive design is content-driven

Start with a one-column mobile layout and add constraints where the content needs them. Breakpoints should represent a layout change, not a catalog of device models.

```tsx
<div className="grid grid-cols-1 gap-4 md:grid-cols-2 xl:grid-cols-3">
  {products.map((product) => <ProductCard key={product.id} product={product} />)}
</div>
```

Avoid dynamically constructing class names such as `` `bg-${color}-500` `` when Tailwind cannot statically detect them. Use an allowlisted map of complete class strings or CSS custom properties. The class map is code, and should be reviewable.

Responsive is more than columns: check navigation, touch target size, dialog height, table strategy, focus visibility, typography wrapping, and the placement of primary actions. Do not solve a dense admin table by shrinking text until it is unreadable.

## 4. Light, dark, and system theme

A theme preference can be `light`, `dark`, or `system`. The system mode follows `prefers-color-scheme`; manual selection takes precedence. Theme preference is not an authentication secret, so local persistence is acceptable, but the durable signed-in preference can be synchronized with the profile API later.

Tailwind v4 can use the system variant by default. For explicit theme switching, define a selector variant:

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));

:root {
  color-scheme: light;
}
.dark {
  color-scheme: dark;
}
```

Use one theme controller at the document root; avoid scattering direct DOM mutations in feature components. To prevent a flash of the wrong theme, initialize before first paint where deployment/CSP policy permits, and test with JavaScript disabled/blocked. Do not put an inline script into production without reviewing the Content Security Policy.

## 5. Accessibility and CSS states

- Preserve a visible keyboard focus outline; never remove it without a stronger replacement.
- Use `:focus-visible` for keyboard-focused controls while keeping pointer interaction clear.
- Ensure status is communicated with text or icon plus text, not color alone.
- Respect `prefers-reduced-motion` for nonessential movement.
- Check contrast in both themes and in disabled/hover/focus states.
- Use CSS logical properties (`margin-inline`, `inset-block`) when they improve internationalization and right-to-left layout.

```css
:focus-visible {
  outline: 3px solid var(--color-brand);
  outline-offset: 3px;
}

@media (prefers-reduced-motion: reduce) {
  html,
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

Treat the reduced-motion rule as a defensive baseline, then replace any meaningful motion with clear static feedback rather than hiding a state change. Avoid global `transition: all`; it animates properties that should not move and can create performance/accessibility problems. Animate only the properties and states that support the interaction.

## 6. Choosing the styling tool

- **Plain CSS:** ideal for foundations, complex selectors, and small projects.
- **CSS Modules:** useful for locally scoped styles in teams that prefer separate stylesheets.
- **Tailwind:** useful for consistent tokenized utility composition and rapid responsive styling, with class readability discipline.
- **Inline styles:** useful for genuinely dynamic values such as a progress percentage; not a replacement for states, media queries, or design tokens.

Do not combine several styling systems without a reason. Choose one default and document the exceptions.

## Debugging lab

1. Reproduce horizontal overflow with a long product name. Find the min-content/grid cause before adding `overflow-x: hidden` to the page.
2. Test the theme while changing the OS setting. Confirm manual light/dark still wins and system mode follows the OS.
3. Navigate with keyboard at 200% zoom in both themes. Find any hidden focus ring, clipped menu, or low contrast.
4. Inspect generated CSS for a dynamic Tailwind class that disappeared from the production build; replace it with a statically detectable class map.

## Exercises

- Convert the catalogue to a fluid grid and document its content-driven breakpoint.
- Add semantic tokens for surface, text, border, danger, focus, and brand. Check both themes.
- Build a mobile admin shell with a collapsible navigation affordance and an accessible name/state.
- Add a theme preference selector with system/light/dark behavior and a no-flash test.
- Compare a small component styled with CSS Modules and Tailwind; write down the maintenance trade-off rather than claiming one universal winner.

## Summary and production tips

**Summary:** CSS is still the layout and rendering system. Tokens encode design decisions; responsive layouts follow content; theming needs explicit precedence; accessibility states are part of the visual system.

```text
semantic HTML → layout primitives → tokens → responsive variants → visual states → measured polish
```

- Avoid magic pixel breakpoints copied from device lists.
- Keep long text and localization in mind when sizing controls.
- Do not let dark mode be “invert the page”; choose semantic colors and test every surface.
- Use browser devtools to inspect actual grid tracks, computed contrast, and overflow.
- Add a style-system dependency only when it solves a real team problem.

## Official documentation

- [Tailwind CSS v4 installation with Vite](https://tailwindcss.com/docs/installation/using-vite)
- [Tailwind theme variables](https://tailwindcss.com/docs/theme)
- [Responsive design](https://tailwindcss.com/docs/responsive-design)
- [Dark mode](https://tailwindcss.com/docs/dark-mode)
- [Hover, focus, and other states](https://tailwindcss.com/docs/hover-focus-and-other-states)
- [Tailwind CSS v4.3 release notes](https://tailwindcss.com/blog/tailwindcss-v4-3)
- [MDN CSS Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout)
- [MDN Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout)
- [MDN media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries)
- [MDN `prefers-reduced-motion`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)

## Readiness criteria

You can choose Grid or Flexbox based on layout; explain a CSS token; build mobile-first UI that survives long content/zoom; implement system and manual theme behavior; and review focus, contrast, and reduced-motion states in both themes.
