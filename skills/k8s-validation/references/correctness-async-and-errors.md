---
name: correctness-async-and-errors
description: Missing await statements, swallowed errors, and silent fallbacks in AI-generated async code.
---

# Async Handling and Error Propagation

> Version: 1.0 | Last Updated: 2026-04-07

These rules catch logic bugs where async operations are mishandled or errors are silently swallowed, leaving the caller with default values that hide real failures.

---

## Rule 1: Async / Await and Promise Handling

**NEVER** omit `await` on async database queries, HTTP calls, or file operations when the result is used synchronously.

Watch for:

- `const result = db.query(...)` without `await` — `result` is a Promise, not a result set.
- `if (fetchUser(id))` where `fetchUser` is async — the Promise is always truthy.
- `.then()` chains that don't return the inner promise, losing the result.
- `try/catch` around an un-awaited async call — errors are not caught because the Promise rejects asynchronously.
- Returning a Promise to a synchronous caller that doesn't `await` it.

**ALWAYS** ensure every async call that produces a needed value is properly awaited or chained.

**Why it matters:** missing `await` is one of the most common AI-generated bugs. The code "looks like" the right pattern, but the values are Promises instead of resolved data, and downstream code silently treats them as truthy or as objects with no useful properties.

---

## Rule 2: Error Handling That Swallows Context

**NEVER** catch errors and silently continue with default or fallback values that hide the real failure.

Watch for:

- `catch (e) { return [] }` — the caller thinks "no results" when the real issue is a database connection failure.
- `try { ... } catch { }` — empty catch blocks that swallow all errors.
- Default values in destructuring that mask missing data: `const { items = [] } = response` hiding a broken API call.
- `|| []` fallbacks that hide undefined responses from a failed call.
- Logging the error but returning success.

**ALWAYS** either:

- Re-throw the error so the caller can handle it, or
- Return an explicit error result (`{ ok: false, error: ... }`) that the caller is forced to check.

**Why it matters:** swallowed errors are how broken systems look healthy. The endpoint returns 200, the dashboard shows zero results, and the real failure is buried in a log line nobody reads. AI assistants love to add fallbacks because the function "feels safer" with one. It isn't.
