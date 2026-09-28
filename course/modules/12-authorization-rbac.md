# Module 12 — Authorization, RBAC, and Permission-Aware UI

> **Build goal:** build a user/admin capability matrix and permission-aware navigation/actions for product CRUD and account administration. Then verify that the same permissions are enforced by the backend and tests—not merely hidden in React.

## 1. Authentication is not authorization

Authentication answers “who is this caller?” Authorization answers “may this principal perform this operation on this resource?” A role is one input into an authorization decision, not a complete policy. Ownership, account state, resource scope, and business rules may also matter.

Example capability matrix:

| Capability | Guest | User | Admin |
|---|---:|---:|---:|
| Read public catalogue | Yes | Yes | Yes |
| Read own profile/orders | No | Yes, own only | Policy-defined |
| Read another user's orders | No | No | Only if business policy permits |
| Create/edit/archive products | No | No | Yes, server-verified |
| Change user role | No | No | Special audited policy |

The backend must check each request and scope database reads/writes to the authenticated principal. A `GET /orders/:orderId` must verify ownership, not just that the caller has a valid session. This prevents IDOR/BOLA bugs.

## 2. Frontend checks improve experience

The UI can avoid presenting actions a person cannot use and can keep navigation understandable. A small capability helper is better than scattering role comparisons:

```ts
type Role = "user" | "admin";
type Capability = "product:read" | "product:write" | "user:manage" | "analytics:read";

const capabilityByRole: Record<Role, readonly Capability[]> = {
  user: ["product:read"],
  admin: ["product:read", "product:write", "user:manage", "analytics:read"],
};

function can(role: Role, capability: Capability): boolean {
  return capabilityByRole[role].includes(capability);
}
```

Use it at the feature boundary:

```tsx
{can(user.role, "product:write") && (
  <Button type="button" onClick={() => openEditor(product.id)}>Edit product</Button>
)}
```

This is only a presentation decision. A malicious user can call the endpoint directly. The API must return 403 and no protected data. Avoid copying a user-controlled role from local storage or query parameters into this helper.

## 3. Deny by default and scope data

- Start with no capabilities; add only explicit permission.
- Enforce resource ownership in the backend query or domain operation, not after sending all records to the browser.
- Check role and object-specific policy on every request, including reads, exports, and mutations.
- Protect role changes, last-admin cases, account deactivation, and sensitive exports with explicit policy and audit logging.
- Don't expose internal account/permission details in error messages if that increases risk.
- A 404 can be used instead of 403 where policy intentionally conceals a resource's existence; the API contract must be consistent.

## 4. HTTP outcomes and user journeys

- **401:** no valid authenticated session. Show sign-in/recovery flow; do not loop.
- **403:** authenticated but not allowed. Show permission-denied UI; do not attempt refresh repeatedly.
- **404:** resource missing or intentionally concealed.
- **409/412:** current state changed or version precondition failed. Refresh/review rather than overwriting.

Permission errors are not generic “network errors.” Don't retry a forbidden operation. Preserve navigation context safely without exposing secrets.

## 5. Testing authorization as a matrix

Frontend tests validate feedback, not server security. API/integration tests must cover role and ownership policy. Use MSW to simulate 401/403 on the frontend, and test the real backend policy in its own suite if available.

| Scenario | Expected UI | Required server assertion |
|---|---|---|
| Guest opens `/admin` | sign-in or safe access-denied path | protected request returns 401 |
| User requests admin product update | no edit affordance; clear 403 state if attempted | response 403; product unchanged |
| User changes order ID in URL | not-found/forbidden state | no other user's order data returned |
| Admin requests allowed product edit | pending/success/conflict UI | server validates role, fields, version, audit |
| Admin changes own role or last admin | clear policy result | special invariant enforced and audited |

## Debugging lab

- Admin action is hidden but request succeeds for normal user: backend authorization bug.
- API returns all users and frontend filters them: excessive data exposure; fix query scope.
- Frontend sees role “admin” in a stale cache after demotion: revalidate auth and clear role-scoped cache; server still rejects every request.
- API returns 401 for an authenticated but unauthorized user: fix status semantics and frontend's refresh behavior.

## Exercises

1. Expand the capability matrix to include notifications, order reads, and analytics.
2. Build an `AdminLayout` UX gate and prove with a crafted request that it is not security.
3. Add tests for 401 versus 403 and ensure 403 never triggers refresh.
4. Write an object-level authorization test for a user attempting another user's order ID.
5. Document audit events and protection for user role changes.

## Summary and production tips

**Summary:** RBAC is useful for organization, but real authorization is a server-side policy over principal, action, resource, and context. The React app reflects that policy; it cannot enforce it.

```text
identity → policy(action, resource, principal) → allow/deny
React affordance → user experience only
```

- Default to deny.
- Scope data before serialization.
- Treat every identifier as attacker-controlled.
- Distinguish 401, 403, not-found, and conflict outcomes.
- Audit privileged changes and guard against last-admin/privilege escalation cases.

## Official documentation

- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [OWASP IDOR Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
- [MDN HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)
- [Northstar backend contract](../capstone/backend-contract.md)
- [React Router route objects](https://reactrouter.com/start/data/route-object)

## Readiness criteria

You can explain why a role check in React is not security, write an allowlist-style capability helper for UX, test both 401/403 outcomes, and describe how the backend prevents IDOR and privilege escalation.
