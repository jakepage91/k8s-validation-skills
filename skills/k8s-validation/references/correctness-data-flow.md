---
name: correctness-data-flow
description: SQL alias mismatches, silently skipped WHERE clauses, and end-to-end data flow gaps in AI-generated query and handler code.
---

# Data Flow and Query Correctness

> Version: 1.0 | Last Updated: 2026-04-07

These rules catch logic bugs where data is read, transformed, or queried in a way that silently produces wrong results. They are some of the most common AI-generated failures because the model pattern-matches on surrounding code without tracing the full flow from request to response.

---

## Rule 1: SQL Column Alias / Application Code Mismatch

**ALWAYS** verify that SQL column names or aliases match the property names used in downstream application code.

Watch for:

- `SELECT column_name ...` where JS or Python code accesses `row.different_name` or `row['different_name']`.
- Missing `AS alias` when the application expects a specific key, e.g. `SELECT conference, COUNT(*)` but the code reads `r.slug`.
- ORM `.select()` or `.raw()` queries where the returned field names don't match destructuring or property access in the consuming code.

**Why it matters:** the mismatch produces `undefined` or `None` values silently. No error, just wrong data: zeros, nulls, empty lists.

---

## Rule 2: WHERE Clause Silently Skipped

**NEVER** build conditional WHERE clauses that silently become no-ops when input is missing.

Watch for:

- `if (param) query += ' WHERE col = ?'` — when `param` is `undefined`, `null`, or `""`, the query returns all rows instead of none or an error.
- ORM patterns like `.where(param ? { col: param } : {})` — an empty filter returns everything.
- Conditional `.filter()` calls that fall through to unfiltered queries.

**ALWAYS** either:

- Validate required filter parameters before querying and return 400 if missing, or
- Explicitly handle the "no filter" case with documented intent (e.g. an admin-only export).

**Why it matters:** missing filters silently return the entire dataset, causing data leakage across tenants, conferences, organizations, or users. This is one of the most dangerous AI-generated bugs because it looks correct in the diff.

---

## Rule 3: Data Flow Across Query Boundaries

**ALWAYS** trace data from input (request params, env vars, config) through processing (queries, transformations) to output (response, UI) to verify the full pipeline is connected.

Watch for:

- A variable is read from the request but never passed into the query.
- A query result field is selected but never included in the API response.
- Middleware sets a value on `req` (e.g. `req.user`) but the handler reads it from a different property (e.g. `req.currentUser`).
- Config values are read at startup but the env var name is misspelled, producing a silent `undefined`.

**Why it matters:** AI-generated handlers often leave dangling intent: the parameter is parsed but never used, the result field is fetched but never returned. The code runs cleanly and the user sees nothing (or stale data) instead of what they asked for.
