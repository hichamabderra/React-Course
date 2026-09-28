# Module 11 — Authentication, Sessions, Cookies, and Client Security

> **Build goal:** implement a realistic sign-in/register/logout/current-user experience against the backend contract. The default model is a server-managed opaque HttpOnly session cookie. React renders authentication-aware UI; the server remains the authority.

## 1. Terms that should not be blurred

- **Identity:** which account the system associates with a request.
- **Authentication:** how the system verifies the caller.
- **Session:** server-recognized continuity across requests.
- **Authorization:** whether that authenticated principal may perform this action on this resource.
- **Frontend auth state:** what the UI currently knows about a session; it can be stale and is not proof accepted by the API.

A user can be authenticated but forbidden. A route can look protected while its API is not. TypeScript types, `isAdmin` client flags, CORS, and hidden buttons do not authorize a server operation.

## 2. Default model: opaque HttpOnly session cookie

On successful login, the server verifies credentials and creates a revocable session. It sets a cookie such as:

```http
Set-Cookie: __Host-northstar-session=<opaque>; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=1800
```

In production `__Host-` requires `Secure`, `Path=/`, and no `Domain` attribute. JavaScript cannot read the HttpOnly value; the browser attaches it to same-origin `/api` requests. The frontend bootstraps state by calling `GET /api/auth/me`.

```ts
async function getCurrentUser(signal?: AbortSignal) {
  return requestJson("/auth/me", parseUserSummary, { signal, credentials: "include" });
}
```

The client must not copy a cookie value into Zustand, React state, localStorage, analytics, URLs, screenshots, or logs. Logout asks the server to revoke the session and clear the cookie; clearing React state alone is not logout.

## 3. Cookie sessions still need CSRF design

Cookies are ambient authority: browsers attach them automatically. An attacker may try to make a victim's browser submit a state-changing request. Use an appropriate `SameSite` policy, validate request `Origin`/`Referer` where appropriate, and implement a CSRF token pattern when the deployment/threat model requires it. SameSite is defense-in-depth, not a complete universal CSRF plan.

CORS controls whether a browser origin may read a response; it is not authentication. Credentialed cross-origin APIs must use an explicit allowed origin, not `*`, and handle credentials/preflight deliberately. Same-origin `/api` is simpler for the capstone.

## 4. Auth bootstrap and UI state

Model loading, authenticated, unauthenticated, and recoverable error states separately. Do not briefly show admin navigation while the session is unknown.

```tsx
type AuthState =
  | { status: "loading" }
  | { status: "signed-out" }
  | { status: "signed-in"; user: UserSummary }
  | { status: "error"; retry: () => void };

function AuthGate({ children }: { children: ReactNode }) {
  const auth = useCurrentUser();
  if (auth.isPending) return <AppShellSkeleton />;
  if (auth.isError && isRecoverableNetworkError(auth.error)) {
    return <SessionCheckError onRetry={auth.refetch} />;
  }
  if (auth.isError && isUnauthorized(auth.error)) return <SignInPrompt />;
  return <>{children}</>;
}
```

In the finished app, represent these with the selected auth/query boundary instead of duplicating identity in several global stores. Avoid treating a temporary network failure as proof the user is signed out; distinguish it from a 401.

## 5. Login, registration, verification, password reset

- Login failures should not reveal whether an email exists. Account verification/disabled states can show a safe next step.
- Registration and password-reset requests use generic responses to reduce account enumeration.
- Email verification and password reset use one-time, expiring server challenges. The frontend never stores a reusable verification secret.
- Browser validation improves usability; the server validates password policy, identity state, rate limits, and all submitted data.
- Rate limits, credential stuffing defenses, MFA, password hashing, session revocation, and audit logs are backend responsibilities—but frontend behavior must respect their status codes.
- After login, validate any `returnTo`/redirect target against same-origin application routes. Never redirect to an arbitrary URL from query input.

## 6. 401, 403, logout, and cached data

- **401:** no valid session for the protected operation. Re-bootstrap or show sign-in; do not loop forever.
- **403:** valid identity but the operation is denied. Do not refresh repeatedly; show permission-denied state.
- On logout or confirmed session revocation, clear auth-scoped Query caches to avoid displaying one user's private data to another session.
- Preserve safe navigation intent where possible; never preserve credentials/form password in route state.

## 7. Alternative: direct SPA access token + HttpOnly refresh cookie

Some deployments use a short-lived access token returned to browser JavaScript and a refresh credential held in an HttpOnly cookie. This differs from the capstone default. If required:

- Keep access token in memory only; XSS can still read it.
- Rotate refresh credentials and detect reuse/replay on the server.
- Protect refresh/logout with CSRF and Origin policy.
- Coordinate simultaneous 401s with a single in-flight refresh; retry the original safe request at most once.
- If refresh fails, clear in-memory auth and sensitive caches, then require sign-in.
- Do not put long-lived tokens in localStorage as a default. Consider a Backend-for-Frontend when it reduces browser token exposure.

## Threat-model checkpoint

| Threat | Frontend contribution | Backend requirement |
|---|---|---|
| XSS/token theft | Avoid unsafe HTML, use CSP/Trusted Types where supported, do not expose long-lived token | Encode/sanitize appropriately, secure session, monitoring, defense in depth |
| CSRF | Same-origin paths, CSRF token integration, no unsafe state-changing GET | SameSite/origin/token verification and appropriate cookie policy |
| IDOR | Do not reveal identifiers as permission; handle 403/404 clearly | Scope each query/action to authenticated principal and resource policy |
| Privilege escalation | Hide unauthorized affordances for UX | Enforce role/capability on every API request and audit privileged changes |
| Account enumeration | Generic sign-up/reset feedback | Rate limit and normalize public responses |

## Debugging lab

- Page says “signed out” after Wi-Fi drops: distinguish network error from a confirmed 401.
- Logout clears the menu but another request still succeeds: server session may not have been revoked.
- A refresh endpoint is called in a loop: stop after one retry; classify 401 vs 403 and clear pending refresh state.
- A protected route works but direct API call does not return 403: backend authorization is missing.
- A cross-origin cookie never arrives: inspect Secure/SameSite/domain/path, `credentials`, CORS headers, and proxy origin; don't weaken policy blindly.

## Exercises

1. Implement auth bootstrap and test loading, 401, offline, active user, and logout states.
2. Add a generic register/reset response and a server field-error mapping.
3. Write a short threat model for session cookie, CSRF, XSS, IDOR, and role escalation.
4. Simulate a revoked session and verify protected cache data is removed.
5. Compare cookie session, BFF, and short access token + refresh cookie; document deployment assumptions and trade-offs.

## Summary and production tips

**Summary:** the browser reflects an authenticated session; the server establishes and revokes it. Cookie flags mitigate some risks, CSRF requires deliberate design, and authorization remains server-side.

```text
login → server verifies → secure cookie
reload → /auth/me → UI identity
API request → server re-authenticates + authorizes
logout → server revokes + clears cookie → client clears private cache
```

- Never store a session secret in a client global store.
- Use HTTPS and Secure cookies in production.
- Do not confuse CORS with permission checks.
- Keep 401, 403, network failure, and account policy states distinct.
- Do not display an order or payment as complete based on optimistic client state alone.

## Official documentation

- [Northstar backend contract](../capstone/backend-contract.md)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [MDN Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [MDN Set-Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [MDN Fetch credentials](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials)
- [MDN CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [OAuth browser-based applications draft/status](https://datatracker.ietf.org/doc/draft-ietf-oauth-browser-based-apps/)

## Readiness criteria

You can distinguish authentication from authorization, explain the cookie/session trust boundary, handle 401 versus 403 and network failure, describe CSRF/CORS/XSS trade-offs, clear private cache on logout, and explain why client route guards are not a security control.
