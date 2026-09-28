# Final Challenge — Product Requirements Document

> **Requirements only. No implementation or reference architecture is included on purpose.** Your first deliverable is an architecture proposal. Do not start by copying a component tree or choosing libraries. Write down your own decisions, trade-offs, security assumptions, and unanswered questions; then request an architecture review before coding.

## 1. Product summary

**Product name:** Northstar Commerce

**Product type:** A responsive commerce storefront with a customer account area and an operations/admin workspace.

**Problem:** Customers need to discover and buy a curated set of products with confidence. Staff need to maintain catalogue and account data, monitor orders, and review business activity. The product must feel coherent across mobile, tablet, and desktop, while the browser remains an untrusted client of a separately operated backend.

**Success outcome:** A customer can browse, search, filter, favorite, manage a cart, place a test order, and review order history. An authorized administrator can maintain products/users and view aggregate analytics. The application presents clear feedback for loading, empty, validation, network, session, permission, and unexpected failure states.

## 2. People and authorization expectations

### Guest

- Browse public pages and the product catalogue.
- Search/filter/sort/paginate the catalogue.
- View product detail and current availability.
- Build a temporary cart/favorites draft before signing in.
- Sign in, register, and recover access.

### User

- View and update their own profile/preferences.
- Add/remove favorites and manage their own cart.
- Place an order using the sandbox checkout flow.
- View only their own order history/details and notifications.
- Sign out and see a clear session-expired path.

### Admin

- View aggregate analytics available to their role.
- Create/update/archive products.
- Search/filter/paginate users and update permitted account roles/status.
- Review operational data permitted by backend policy.
- Receive clear feedback if an operation is denied, conflicts with another edit, or fails validation.

**Authorization requirements:** A route guard, hidden button, disabled form, or client-side role check is a convenience only. The API must enforce ownership and role policy for every request. The UI must distinguish unauthenticated (`401`) from authenticated-but-forbidden (`403`) states. The system must deny by default and must not trust a role included in a request body.

## 3. Functional requirements

### 3.1 Public storefront

1. A visitor can load a public landing page with a clear entry into the catalogue.
2. The catalogue displays product name, image/alternative text, category, price/currency, availability, and a route to product detail.
3. Search matches the supported product fields and communicates when no results match.
4. Category filters, sort order, and page/page-size behavior are explicit and work on narrow screens.
5. Search, filters, sorting, and pagination can be linked/bookmarked and restored after refresh/back/forward navigation.
6. Changing one filter does not leave the user on an invalid page index.
7. The product detail page shows the current product data, stock availability, and a meaningful loading, not-found, or error state.
8. Images have an appropriate text alternative when informative; decorative imagery is not announced as content.

### 3.2 Authentication and account

1. Registration, login, logout, current-user bootstrap, email-verification concept, and password-reset concept have documented request/response behavior.
2. The product does not store a long-lived sensitive credential in browser local storage.
3. A refresh/reload of an authenticated browser session restores the correct user state without exposing an HttpOnly credential to JavaScript.
4. An expired/invalid session is handled once, safely, and without an infinite redirect/refresh loop.
5. The profile form supports field-level errors, server validation errors, pending/submission states, and success feedback.
6. Account settings include a theme preference and notification preference where supported by the API.
7. A user cannot view another user's profile or orders by changing an identifier in the URL.

### 3.3 Cart, checkout, and orders

1. A guest can stage a cart using product IDs and quantities only; it contains no credentials, payment data, address secrets, or authoritative prices.
2. After sign-in, the guest draft is reconciled with the server cart. The server validates current availability, price, and quantity and returns the canonical cart.
3. Authenticated cart changes are persisted by the API and the UI reconciles against the server response.
4. The cart clearly communicates changed availability/price and prevents checkout with invalid quantities.
5. Favorite toggles and cart quantity changes may show an optimistic response when that improves perceived latency; on rejection they must reconcile/roll back and explain the authoritative server result. Payment authorization and order completion are never presented as successful before the server confirms them.
6. Checkout uses a payment-provider sandbox/token, not raw card numbers in the React application.
7. Order submission is safe against accidental duplicate submission/retry using the API's idempotency contract.
8. The backend calculates totals, tax, shipping, discount eligibility, and stock reservation; a client-provided total is never trusted.
9. Users can see their own order list, order details, and order state; unauthorized identifiers do not disclose another user's data.

### 3.4 Customer experience

1. The account area provides profile, settings, favorites, cart, orders, and notification entry points.
2. Notifications distinguish read/unread state and announce meaningful updates accessibly.
3. Customer pages support empty states (no favorites/orders/notifications) with a useful next action.
4. Destructive actions are clearly labelled and require appropriate confirmation; cancel remains possible.

### 3.5 Admin operations

1. The admin area contains product management, user management, and analytics pages.
2. Product management supports create, edit, archive/delete according to the backend contract, server validation, pending state, success feedback, and conflict handling.
3. User management supports search/filter/sort/pagination and role/status changes allowed by server policy.
4. Admin lists communicate total/current page and work responsively; mobile views must not require inaccessible horizontal scrolling for every field.
5. Analytics have clear time range, definitions, loading/error/empty states, and do not expose personally identifiable details without a defined permission.
6. Admin actions are audited server-side. The UI never implies that hiding an action makes an API operation secure.

### 3.6 Forms and validation

1. Login, registration, profile, product create/edit, checkout, and admin user forms have labels, instructions where needed, accessible field errors, pending/disabled states, and success/error feedback.
2. Client validation gives fast feedback; the server independently validates every request.
3. Validation errors are associated with the relevant fields and announced to assistive technology.
4. When a server rejects a form, safe user input is preserved where appropriate; secrets are cleared when policy requires it.
5. Any asynchronous uniqueness/availability check is cancellable/debounced and cannot overwrite a newer value's result.

### 3.7 Interaction quality

1. The application provides responsive page layouts, mobile navigation, mobile-safe dialogs, and tables suitable for small screens.
2. The user can navigate all interactive journeys by keyboard, including menus, dialogs, forms, tables, and notifications.
3. Focus is moved appropriately when dialogs open/close, routes change, validation fails, or an operation completes.
4. Dark/light/system theme behavior is consistent and does not reduce contrast.
5. Animations are brief, purposeful, interruptible where appropriate, and respect `prefers-reduced-motion`.
6. Loading, error, empty, and success states are deliberately designed; a spinner is not the only loading strategy.

## 4. Non-functional and quality requirements

### Security and privacy

- Use secure transport in production and a deliberate session/CSRF/CORS policy.
- Keep server secrets out of the client bundle and public environment variables.
- Avoid unsafe HTML insertion. If rich text is required, define sanitization and Trusted Types/CSP policy.
- Keep authentication/session data out of logs, URLs, analytics, query keys, and screenshots.
- Respect data minimization: expose only the data required for the task and role.
- The backend enforces authentication, authorization, ownership, rate limiting, schema validation, and business rules.
- Document account/session revocation behavior and user-facing expired-session recovery.

### Accessibility

- Target WCAG 2.2 AA as a product quality goal, with semantic HTML, labels, keyboard operation, visible focus, meaningful names, contrast, and error identification.
- Use automated checks as one signal, not as proof. Include manual keyboard/zoom/screen-reader review in release acceptance.
- The application remains usable at narrow widths and at browser zoom.

### Performance

- Establish a reproducible baseline before optimizing.
- Define the route-level JavaScript and image budgets from the selected deployment and expected device/network profile.
- Measure core journeys with browser performance tooling and appropriate Core Web Vitals thresholds from current official guidance.
- Avoid downloading or rendering data the current view does not need. Use server pagination for large admin data sets.
- Add virtualization only if observed list/render costs warrant its complexity.

### Reliability and observability

- Every request path has a user-understandable failure state and a request/correlation ID path for support.
- Retry only safe/retryable operations; respect rate limits and idempotency.
- Log structured operational context on the server without sensitive values.
- Add a production build, typecheck, lint, automated tests, and smoke test to CI.
- Document environment configuration, deployment routing, cache invalidation, and rollback.

### Browser and responsive support

- Support current stable evergreen desktop and mobile browsers defined by the product team.
- Validate mobile, tablet, and desktop breakpoints; do not validate only with a desktop screenshot.
- Essential workflows remain usable when a network request is slow or fails.

## 5. Error and state acceptance

For each major page/request, specify and test the states below:

- **Initial/pending:** useful structure is visible; repeated actions are prevented or safely deduplicated.
- **Background refresh:** existing valid data can remain visible with a clear refreshing indicator where appropriate.
- **Success:** outcome is visible and focus/announcement behavior is reasonable.
- **Empty:** absence of data is distinct from an error and offers an appropriate next step.
- **Validation:** field-level issues are connected to fields; server and client messages do not conflict.
- **Unauthenticated:** `401` routes to a safe sign-in journey without losing intended navigation where supported.
- **Forbidden:** `403` communicates lack of permission; it does not repeatedly refresh the session.
- **Not found/conflict:** missing resources and concurrent edits are handled without accidental data loss.
- **Network/server failure:** user can safely retry; errors do not leak internal details.

## 6. Product-level acceptance scenarios

Use these as observable behavior, not component implementation requirements.

1. **Search and URL restoration:** A shopper searches “headphones”, selects Audio, sorts by price, opens a product, returns, and sees the same catalogue state. Copying the URL into a fresh tab restores the same filter/sort/page state.
2. **Empty catalogue result:** A query with no matching products returns a designed empty state and a way to clear filters. It is not shown as an API failure.
3. **Guest cart merge:** A guest adds two products, signs in, and sees the server-confirmed merged cart. A product whose stock changed is explained and not silently charged at a stale price.
4. **Order retry:** A request times out after submission; the user retries safely and does not create a duplicate order.
5. **Profile validation:** An invalid email/name receives inline, announced feedback; a server-side rejection is rendered without losing safe input.
6. **Permission denial:** A normal user manually navigates to an admin URL and sends an admin API request. The UI gives a sensible state and the API returns `403`; no protected data is returned.
7. **Session expiration:** An authenticated request returns `401`. The app recovers once according to the selected session model, then signs out safely if the session is revoked.
8. **Admin concurrent edit:** An admin opens a product, another admin edits it, and the first submit receives a conflict/precondition failure. The UI prevents a silent overwrite and offers reload/review.
9. **Mobile admin table:** A user can find a person/product and perform an allowed operation without relying on pointer-only interactions or an unusable wide table.
10. **Reduced motion:** With OS reduced-motion enabled, page/dialog transitions remain understandable without nonessential motion.

## 7. Delivery constraints and non-goals

- This is a frontend capstone with a documented backend contract. A real backend is optional during learning; MSW supplies deterministic development/test responses.
- Payment uses a provider sandbox or simulated provider flow. No raw payment-card data is collected or stored.
- Email delivery, password hashing, database schema, billing integration, inventory reconciliation, and production infrastructure are backend/team responsibilities, but their contracts and security implications must be understood by the frontend engineer.
- Do not add a library solely because it appears in the roadmap. Record the problem it solves and the ongoing cost it introduces.
- Do not build every admin analytic metric without a definition and data source.
- Do not treat client-side validation, route guards, TypeScript types, CORS, or hidden controls as an API security boundary.

## 8. Your architecture proposal — submit before implementation

Create a Markdown proposal from this PRD only. Do **not** write the application yet. Include:

1. Product assumptions and questions that need stakeholder/backend answers.
2. A route/page map, including public, user, admin, error, and not-found experiences.
3. A state ownership table for URL/local/form/server/global state and why each value belongs there.
4. A proposed feature/module boundary and dependency direction; explain how it can grow without creating empty folders.
5. A data-flow diagram from browser to API and back for catalogue, profile, cart, and admin update.
6. Authentication/session choice, cookie/CSRF/CORS assumptions, and a threat model for XSS, token theft, IDOR, and privilege escalation.
7. An RBAC capability matrix for User and Admin, with explicit backend enforcement points.
8. API gaps or response semantics you would clarify before coding.
9. A testing strategy: what is unit-tested, component/integration-tested, MSW-mocked, and exercised in Playwright.
10. Accessibility, responsive, performance, error, observability, and deployment acceptance criteria.
11. Two alternatives you considered and why you did not choose them.

**Review gate:** ask for an architecture review before implementation. The review should challenge ownership boundaries, trust boundaries, route/data duplication, auth assumptions, accessibility, and operational risks. After that review, build in small vertical slices, write tests with the behavior, and update the decision log when evidence changes your design.

## Official references

- [React Router modes](https://reactrouter.com/start/modes)
- [React Router v8 upgrade notes](https://reactrouter.com/upgrading/v7)
- [TanStack Query](https://tanstack.com/query/latest/docs/framework/react/overview)
- [MSW](https://mswjs.io/docs/)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [MDN: Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
