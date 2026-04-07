---
name: correctness-http-and-types
description: HTTP method/parameter source mismatches, environment-dependent code paths, and type coercion bugs in AI-generated request handlers.
---

# HTTP Method, Parameter Sources, and Type Handling

> Version: 1.0 | Last Updated: 2026-04-07

These rules catch logic bugs where the AI wrote code that "looks right" but reads parameters from the wrong place, makes assumptions about runtime types, or comparing values across system boundaries. They are silent failures: no error, just wrong data.

---

## Rule 1: HTTP Method / Parameter Source Mismatch

**NEVER** read parameters from a source that doesn't match the HTTP method.

- `GET` / `DELETE` requests: parameters come from `req.query` (Express), `request.args` (Flask), query string parsers, or URL path params. **Not** `req.body` / `request.json`.
- `POST` / `PUT` / `PATCH` requests: parameters come from `req.body` / `request.json` / form data.

Watch for:

- `req.body` in a `router.get()` or `app.get()` handler.
- `request.json` in a `@app.route(..., methods=['GET'])` handler.
- `req.query` in a `router.post()` handler when body params were intended.

**Why it matters:** the handler silently receives `undefined` / `None` for every parameter, causing filters and conditions to be skipped. No error is thrown — the code just returns wrong results.

---

## Rule 2: Type Coercion and Comparison Bugs

**ALWAYS** use strict equality and explicit type conversion at system boundaries.

Watch for:

- `==` instead of `===` in JavaScript, e.g. `id == req.params.id` where one is a number and one is a string.
- Comparing database integer IDs with string URL params without parsing.
- `Boolean("false") === true` in JS — the string `"false"` is truthy.
- Python `if value:` when `value` could be `0` or `""` (both falsy but valid).
- Comparing dates without parsing them to a common type first.

**Why it matters:** string-vs-number comparisons silently produce wrong results. The function runs, the response is returned, and the data is wrong.

---

## Rule 3: Environment-Dependent Code Paths

**NEVER** assume runtime environment matches development assumptions.

Watch for:

- Code that works with in-memory or SQLite DB but fails on Postgres/MySQL (different type coercion, case sensitivity, JSON handling).
- Relying on object key ordering (not guaranteed in JS for numeric keys).
- Locale-dependent string comparisons or date parsing.
- File paths that assume a specific OS (hardcoded `/` vs `\`).
- Code that works against a local mock but breaks against the real upstream because the real one returns slightly different field shapes.

**Why it matters:** these bugs only show up when the code meets the real environment. They're invisible in unit tests and reviews.
