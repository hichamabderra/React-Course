# Module 16 — Motion, Interaction Feedback, and Reduced Motion

> **Build goal:** add subtle enter/exit and layout feedback to a drawer, product favorite, and route transition only where it clarifies change. Respect `prefers-reduced-motion` and keep the app usable with animations removed.

## 1. Motion should explain state

Animation can show continuity, confirm an action, reveal hierarchy, or preserve spatial context. It can also delay tasks, distract, cause motion discomfort, or make performance worse. Begin with the purpose, not a library API.

Use native CSS for simple hover/focus/color transitions. Use Motion when orchestration, gestures, enter/exit presence, or layout transitions provide a meaningful benefit that would otherwise be complex.

## 2. Motion for React v13

The stack uses the `motion` package **13.4.4**; import React APIs from `motion/react`.

```tsx
import { motion } from "motion/react";

function FavoriteButton({ saved }: { saved: boolean }) {
  return (
    <motion.button
      type="button"
      aria-pressed={saved}
      whileTap={{ scale: 0.96 }}
      transition={{ type: "spring", stiffness: 500, damping: 30 }}
    >
      {saved ? "Saved" : "Save to favorites"}
    </motion.button>
  );
}
```

The button's state and accessible name remain understandable without animation. `whileTap` is feedback, not the only indication that a change occurred. Some current Motion accessibility docs still show the older `framer-motion` import; translate to `motion/react` for this pinned package and follow the current changelog.

## 3. Enter/exit and layout

A dialog/drawer may need exit animation before removal. `AnimatePresence` observes children leaving the React tree and enables their exit animation; it does not replace the dialog primitive's focus/keyboard behavior.

```tsx
import { AnimatePresence, motion } from "motion/react";

<AnimatePresence>
  {open && (
    <motion.aside
      key="cart-drawer"
      initial={{ opacity: 0, x: 24 }}
      animate={{ opacity: 1, x: 0 }}
      exit={{ opacity: 0, x: 24 }}
      transition={{ duration: 0.18 }}
    >
      <CartContents />
    </motion.aside>
  )}
</AnimatePresence>
```

The exit animation may keep DOM mounted briefly; ensure the component does not remain interactive to assistive technology after close. A well-built Dialog primitive owns modal semantics; coordinate animation with it and verify focus/background behavior.

React's `<ViewTransition>` is another current option for coordinated transitions with React transitions/Suspense. Avoid combining two animation owners on the same element; choose either Motion layout animation or React ViewTransition for that interaction and test browser support.

## 4. Reduced motion

Respect the operating-system preference by removing or replacing nonessential spatial movement. Motion's `MotionConfig` can automatically reduce transform/layout animation; `useReducedMotion` supports custom behavior.

```tsx
import type { ReactNode } from "react";
import { MotionConfig } from "motion/react";

function AppProviders({ children }: { children: ReactNode }) {
  return <MotionConfig reducedMotion="user">{children}</MotionConfig>;
}
```

Keep essential state changes understandable when motion is removed. Do not replace a loading state with a moving skeleton that has no static alternative. Check reduced-motion with the browser/OS setting, not only a code branch.

## 5. Performance and motion design

- Prefer transform/opacity over layout-affecting properties when appropriate; profile rather than assume.
- Avoid large, continuous, scroll-linked movement without a clear purpose.
- Avoid animation that delays an urgent action or traps focus.
- Avoid animating every list item on every server refresh.
- Keep durations short for routine feedback and provide a way to stop long-running motion.
- Be aware that layout animations can cause measurement work and interact with responsive layouts.
- Respect reduced motion in both CSS and JS animation systems.

## Debugging lab

- Exit animation never runs: the item was replaced/re-keyed or `AnimatePresence` does not see the conditional child.
- Focus remains in an invisible drawer: focus/modal lifecycle is not coordinated with exit; fix the primitive flow.
- Reduced-motion mode still moves the entire page: configure Motion and inspect CSS animations separately.
- List reorder feels expensive: animate only meaningful item changes and profile the layout work.

## Exercises

1. Add a short favorite confirmation that preserves text/`aria-pressed` meaning.
2. Animate a drawer enter/exit while preserving focus behavior.
3. Test system reduced-motion and document which animations are disabled versus replaced by opacity.
4. Compare Motion layout animation with React ViewTransition for one small route or shared element; pick one and justify it.
5. Remove all motion from the page. Verify the user still understands what changed and can complete the workflow.

## Summary and production tips

**Summary:** motion is an optional communication layer over real state. It must not be the only feedback, a barrier to interaction, or a substitute for correct accessible primitives.

```text
state change → semantic feedback first → purposeful animation → reduced-motion alternative
```

- Use CSS when CSS is sufficient.
- Prefer accessible names and state attributes that remain true without animation.
- Keep reduced-motion in acceptance tests.
- Avoid duplicate animation systems on one element.
- Consult current Motion docs; package/import names changed from `framer-motion` to `motion`.

## Official documentation

- [Motion for React](https://motion.dev/docs/react)
- [Animation](https://motion.dev/docs/react-animation)
- [AnimatePresence](https://motion.dev/docs/react-animate-presence)
- [Layout animations](https://motion.dev/docs/react-layout-animations)
- [Gestures](https://motion.dev/docs/react-gestures)
- [Motion accessibility](https://motion.dev/docs/react-accessibility)
- [useReducedMotion](https://motion.dev/docs/react-use-reduced-motion)
- [React ViewTransition](https://react.dev/reference/react/ViewTransition)
- [MDN prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)
- [WCAG animation from interactions](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html)

## Readiness criteria

You can articulate why each animation exists, distinguish state from transition, make a reduced-motion alternative, preserve dialog focus behavior, and decide when native CSS or no animation is the better production choice.
