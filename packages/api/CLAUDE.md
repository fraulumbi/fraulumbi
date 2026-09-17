# API conventions

Applies to everything under `packages/api/`. Inherits the repo-wide rules in
`.claude/CLAUDE.md` — this file only adds what's specific to the HTTP layer.

## Endpoint naming

- Resources are **plural nouns**, never verbs: `/orders`, not `/getOrders`.
  The HTTP method already carries the verb.
- Paths are `kebab-case` and lowercase: `/purchase-orders`, not
  `/purchaseOrders` or `/purchase_orders`.
- Hierarchy reflects ownership, and nests at most two levels deep:
  `/orders/{order_id}/line-items`. Past two levels, expose the child as a
  top-level resource with a filter instead: `/line-items?order_id=...`.
- Path params are the resource's identifier and nothing else. Everything that
  narrows a collection is a query param: filtering, sorting, pagination.
- Method semantics are not negotiable:
  - `GET` — safe and idempotent, never mutates, never has a request body
  - `POST` — creates a subordinate resource, or a non-idempotent action
  - `PATCH` — partial update; absent fields are left untouched
  - `PUT` — full replacement; absent fields are reset to their default
  - `DELETE` — idempotent; deleting an already-deleted resource returns `204`
- Actions that genuinely aren't CRUD get a sub-path verb, used sparingly:
  `POST /orders/{id}/void`, `POST /shipments/{id}/manifest`.
- Every path is versioned at the prefix: `/api/v1/orders`. Never version with a
  header or a query param.
- Collections are paginated by default. Never return an unbounded list.

## Request schemas

- JSON only. Field names are `snake_case`, consistent with the database columns
  they map to.
- Validate at the boundary, before any business logic runs. A request that
  reaches a context/service function has already been proven well-formed.
- Reject unknown fields with `422` rather than ignoring them — silently dropping a
  misspelled field is how bugs ship.
- Types are explicit and strict: no coercing `"5"` to `5`, no accepting `"true"`
  for a boolean. Timestamps are ISO 8601 with an offset (`2026-09-17T14:03:00Z`).
  Money is an integer count of minor units plus a currency code — never a float.
- Identifiers in a payload are the public ID, never an internal autoincrement.
- `POST` endpoints that create resources accept an `Idempotency-Key` header and
  return the original result on a repeat.

## Response schemas

- Success responses wrap the payload in a `data` key so the envelope can grow
  without a breaking change:
  ```json
  { "data": { "id": "ord_01H...", "status": "fulfilled" } }
  ```
- Collections return `data` as an array alongside a `meta` object carrying
  `next_cursor` and, only when it's cheap to compute, `total_count`.
- Status codes are used precisely: `200` read/update, `201` create with a
  `Location` header, `202` accepted-but-async, `204` delete, `400` malformed,
  `401` unauthenticated, `403` unauthorized, `404` missing, `409` conflict,
  `422` semantically invalid, `429` rate-limited.
- Errors share one shape across every endpoint:
  ```json
  {
    "error": {
      "code": "license_expired",
      "message": "The license on file expired on 2026-08-01.",
      "field": "license_number"
    }
  }
  ```
  `code` is a stable `snake_case` string clients may branch on. `message` is for
  humans and may be reworded freely. Never leak a stack trace or a raw SQL error.
- Validation failures return `422` with an array of those error objects, one per
  offending field — not just the first failure.
- Field presence is stable. A nullable field is always present with `null`; it is
  never omitted. Adding a field is backwards-compatible, removing or renaming one
  is not and requires a new version.
- Response shape for a given resource is identical whether it was returned from a
  create, an update, or a read.

## Checklist for a new or changed endpoint

- [ ] Path, method, and status codes match the rules above
- [ ] Request validated at the boundary; unknown fields rejected
- [ ] Authorization checked server-side, scoped to the caller's company
- [ ] Collection endpoints paginated and bounded
- [ ] Error paths return the shared error shape, with no internals leaked
- [ ] Changes are additive, or shipped behind a new version prefix
- [ ] Tests cover one success case and at least one `4xx` case
- [ ] API docs/OpenAPI spec updated in the same PR
