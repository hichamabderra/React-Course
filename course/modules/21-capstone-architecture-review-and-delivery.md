# Module 21 — Independent Capstone Delivery and Architecture Review

> **This module deliberately does not provide an implementation or reference architecture.** The final challenge ends with requirements and an architecture proposal. The learner makes the design decisions, records trade-offs, and requests review **before** coding.

## Build goal

Deliver a reviewed proposal for the Northstar Commerce PRD and, only after review, implement the product in small vertical slices using the course concepts. The graded artifact is not a particular folder tree or library combination; it is evidence that the design meets requirements, respects trust boundaries, and can be tested and maintained.

## First principles: architecture is a set of testable decisions

Architecture is not a folder diagram chosen before understanding the product. It is a set of decisions about ownership, trust, dependencies, change, and verification. Requirements constrain those decisions; assumptions expose uncertainty; alternatives make trade-offs visible. A proposal is a hypothesis to review, not a verdict to defend at all costs.

An architecture review before implementation reduces the cost of discovering high-impact mistakes—such as trusting client roles, duplicating remote data, or losing checkout idempotency—without requiring the course to reveal a preferred implementation. After review, the learner still chooses and owns the design.

## The challenge sequence

### Gate 1 — Read the requirements

Read [Final Challenge PRD](../capstone/final-challenge-prd.md) and [Backend Contract](../capstone/backend-contract.md). Mark each requirement as:

- user-visible behavior;
- data/API contract;
- security/privacy policy;
- accessibility/performance/operational quality;
- an assumption or unresolved question.

Do not start by coding components or adding packages.

### Gate 2 — Submit your architecture proposal

Produce the requested route/page map, state ownership table, dependency boundaries, data-flow diagrams, auth threat model, RBAC capability matrix, testing plan, accessibility plan, performance budget, and deployment assumptions. Record at least two alternatives and why you rejected them.

Ask a reviewer to challenge:

- whether URL/server/form/local/global values have one clear owner;
- whether authorization is enforced outside the browser;
- whether routes and API requests expose private records;
- whether guest-cart merge, stale stock, and order retry are safe;
- whether failures and concurrent edits recover without silent data loss;
- whether keyboard/screen-reader journeys are possible;
- whether dependencies and caching/retry policies are justified.

**Do not receive or request a solution implementation before this review.** The purpose is to practice making decisions under requirements, not matching a reference tree.

### Gate 3 — Build vertical slices

After feedback, implement the smallest working slice that crosses the relevant UI, API contract/mock, error state, accessibility, and test boundary. For example, a slice may include route → query → pending/error/empty/success → keyboard test. Keep each pull request reviewable and record evidence when a design decision changes.

### Gate 4 — Release review

Before delivery, run the course quality gates, tests, mobile/keyboard/screen-reader checks, bundle/performance review, dependency/security review, production build, deep-link smoke test, API/session checks, and rollback rehearsal. Record known limitations rather than hiding them.

## Submission artifacts

1. Architecture proposal and diagrams (pre-implementation version retained).
2. Decision log with context, options, choice, consequences, and revisit signals.
3. Implemented vertical slices and automated test evidence.
4. Threat model and user/admin capability matrix.
5. Accessibility review findings and remediations.
6. Performance baseline and at least one evidence-based optimization decision.
7. Deployment/release checklist with monitoring and rollback notes.
8. A short retrospective: which assumption changed, which abstraction was removed, and what you would do differently with a real backend/team.

## Review rubric — assess evidence, not a prescribed design

| Area | Strong evidence |
|---|---|
| Product requirements | Every required user journey maps to a route/state/test; assumptions are explicit. |
| Architecture | Boundaries have owners and dependency direction; state is not duplicated without a reason. |
| Security | Session, CSRF/XSS, RBAC, ownership, and secret handling are explicit; client checks are never called security. |
| Data behavior | API types are runtime-validated; errors, pagination, optimistic rollback, conflict, and idempotency are handled. |
| Accessibility | Core tasks work by keyboard; names/labels/focus/announcements are reviewed; automated results have manual follow-up. |
| Reliability | Tests cover behavior and failure paths; safe retries and correlation IDs are documented. |
| Performance | Baseline and budget are defined; changes are measured; unnecessary memoization/virtualization is avoided. |
| Delivery | Quality gates, deployment routing/cache, smoke tests, monitoring, and rollback are reproducible. |
| Reasoning | Alternatives and trade-offs are explained; design changes respond to evidence. |

## Debugging an architecture review

| Review symptom | Questions to investigate before changing the design |
|---|---|
| The same product appears in local state, a loader result, and Query | Which owner is canonical? Are these separate lifecycles or duplicated copies? Can one cache entry serve the route and component? |
| The admin screen looks protected but a user can call its API | Which server policy is missing? What should the user see for 401 versus 403? Which direct-request test proves the fix? |
| Guest cart merge overwrites server quantities or prices | Which values are proposals versus authority? What conflict/result does the API return? How is reconciliation shown? |
| A failed mutation silently loses form input | Which data is safe to preserve? Where are field/domain errors mapped? What should be rolled back versus reloaded? |
| A “fast” architecture adds multiple libraries and caches | Which measured requirement does each tool satisfy? What is its maintenance cost and removal signal? |
| Screen-reader or keyboard flow is unclear from the proposal | What is the task, focus sequence, accessible name/status, and manual test evidence? |

Treat review comments as evidence requests. Reproduce the risk, update the decision log, and revise only the boundary that fails the requirement.

## Exercises before implementation

1. Convert every PRD acceptance scenario into a requirement-to-route/state/test trace row.
2. For each state value, name one owner and one reason it does not belong in the nearest alternative owner.
3. Ask a reviewer to role-play a normal user editing an admin URL and a guest retrying an order after a timeout; record the expected API behavior and UI message.
4. Write two architecture alternatives, including their operational or security costs, and a condition that would make you revisit the selected option.
5. Review your proposal using the rubric without adding any implementation code. Mark unknowns as questions rather than silently inventing backend behavior.

## Summary, mental model, and production tips

**Summary:** requirements define observable outcomes; architecture records ownership and trust decisions; review challenges assumptions; vertical slices turn approved decisions into evidence. There is intentionally no single prescribed component tree or reference implementation.

```text
PRD → assumptions + questions → proposal + alternatives → review → small vertical slices → tests + release evidence
```

- Keep the pre-review proposal so the retrospective can distinguish original assumptions from later evidence.
- Prefer a small explicit decision over a generic framework built for hypothetical future needs.
- Record who owns every sensitive or remote value and where enforcement occurs.
- Revisit an architecture choice when requirements, measurements, team shape, or backend contracts change.
- Do not begin implementation until the required architecture review is complete.

## Final reflection prompts

- Which state was hardest to assign an owner to, and what evidence resolved it?
- Which UI behavior did you first model as an Effect but later derive or handle in an event?
- Which operation did you refuse to make optimistically successful, and why?
- What did you discover about the difference between a route guard and authorization?
- Which performance optimization did you decide **not** to ship?
- What would change if this application needed server rendering, offline support, or multiple frontend teams?

## Official references

- [Final Challenge PRD](../capstone/final-challenge-prd.md) — requirements only.
- [Course Guide](../README.md) — learning outcomes, stack, architecture assumptions, and roadmap.
- [Backend Contract](../capstone/backend-contract.md) — API and security flows.
- [Verified Stack](../verified-stack.md) and [Official Documentation Map](../official-documentation-map.md).
- [GitHub pull request review guidance](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests).

## Readiness criteria

The architecture proposal exists before implementation and is reviewable; every major requirement has an owner and verification plan; trust boundaries are explicit; release criteria are testable; and the learner can defend trade-offs without relying on a reference implementation.
