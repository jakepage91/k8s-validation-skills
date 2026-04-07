---
name: correctness-api-contracts
description: Response shape, pagination shape, and consumer contract mismatches in AI-generated API endpoints.
---

# API Contracts and Response Shapes

> Version: 1.0 | Last Updated: 2026-04-07

These rules catch logic bugs where the backend and the consumer (frontend, another service, an SDK) silently disagree about what an endpoint returns. They show up as empty UIs, zero counts, broken pagination, and "I added it but nothing happens."

---

## Rule 1: Response Shape / Consumer Contract Mismatch

**ALWAYS** verify that the shape of API responses matches what consumers (frontend code, other services) actually destructure or access.

Watch for:

- Backend returns `{ items: [...] }` but frontend reads `response.data.results`.
- Endpoint returns a flat array but the consumer expects `{ data: [...], total: N }`.
- Field renamed on the backend (`name` → `title`) without updating all consumers.
- Pagination response missing `total`, `next_page`, or `has_more` that the frontend relies on.

**Why it matters:** shape mismatches cause silent failures. Components render empty, counts show 0, pagination breaks. There is no error in the network tab, just the wrong data shape, and the frontend's optional chaining (`response?.items?.length`) hides the problem.

---

## Rule 2: Pagination and Boundary Errors

**ALWAYS** verify pagination math: `offset + limit` logic, zero-based vs one-based page numbers, and edge cases like empty result and last page.

Watch for:

- `offset = page * limit` when `page` is 1-based — skips the first page of results.
- `LIMIT` and `OFFSET` swapped in SQL.
- Array `.slice(start, end)` where `end` is exclusive but treated as inclusive.
- Missing `Math.ceil` in total page count calculation.
- Off-by-one errors in cursor-based pagination where the cursor is included or excluded inconsistently.

**Why it matters:** pagination bugs are easy to miss in code review because the function looks correct. They only show up when a real user clicks "next" and sees the wrong results.

---

## Rule 3: Backwards-Incompatible Changes Without Versioning

**NEVER** rename fields, change types, or remove fields from an API response without either versioning the endpoint or updating every known consumer.

Watch for:

- Renaming a JSON field from `created_at` to `createdAt` in the response.
- Changing a field type from `string` to `object`.
- Removing a field that the frontend or other services depend on.

**Why it matters:** the AI doesn't know who consumes your API. A "small cleanup" of a field name can break a dashboard, a mobile app, or a downstream service. Treat every API response shape as a contract.
