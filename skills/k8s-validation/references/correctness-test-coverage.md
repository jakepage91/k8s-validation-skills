---
name: correctness-test-coverage
description: Rules requiring AI-generated code to ship with tests that exercise the new behavior, not just the happy path.
---

# Test Coverage for AI-Generated Code

> Version: 1.0 | Last Updated: 2026-04-07

These rules require that AI-generated code arrives with tests that actually exercise its behavior. They exist because the easiest way for AI-generated code to look correct and be wrong is to ship without a test that proves it works against real input.

---

## Rule 1: New Endpoints and Handlers Require Integration Tests

**ALWAYS** when generating a new HTTP handler, message consumer, or background job, also generate at least one integration test that:

- Sends a realistic input through the handler.
- Asserts on the response shape and the database (or other side-effect) state.
- Covers at least one error case (invalid input, missing field, unauthorized).

Watch for:

- AI generated a `/summarise` endpoint with no test in `tests/integration/`.
- AI generated a webhook handler with a happy-path unit test but no test for invalid signatures.
- AI generated a queue consumer with no test that exercises a real message.

**Why it matters:** if there is no test, the code's correctness is a guess. Integration tests run against real dependencies (or as close as you can get) are the only thing that proves the new code does what it claims, not what the AI hoped it would do.

---

## Rule 2: Tests Must Exercise the Specific Risk

**ALWAYS** when adding security or filtering logic, write a test that proves the filter blocks the thing it claims to block.

Watch for:

- A `filter_pii` function with no test that sends actual PII.
- An auth middleware with no test that sends an unauthenticated request.
- A path traversal guard with no test that sends `../../../etc/passwd`.
- A rate limiter with no test that sends N+1 requests.

**Why it matters:** AI assistants love generating "the function exists." That is not the same as "the function works." A regex-based PII filter that only matches `\d{3}-\d{2}-\d{4}` is structurally correct and behaviorally broken. The only way to know is to send the input.

---

## Rule 3: Tests Must Run Against Real Dependencies Where Possible

**ALWAYS** prefer integration tests that hit real (or staging) dependencies over unit tests with mocks for AI-generated code.

Watch for:

- Mocked database tests for code that does non-trivial SQL.
- Mocked HTTP client tests for code that integrates with a third-party API.
- Mocked Kubernetes client tests for code that talks to the cluster.

**Why it matters:** the most common failure mode for AI-generated code is "the code is reasonable in isolation but wrong against the real environment." Mocks reproduce the AI's assumptions, not reality. Integration tests against staging (e.g. via mirrord) are how you find out the env var is misnamed, the database column has a different type, or the upstream API returns a slightly different shape than the AI expected.

---

## Rule 4: Generated Tests Must Actually Run

**ALWAYS** ensure that newly generated test files are picked up by the test runner.

Watch for:

- New test file in a directory not covered by `pytest.ini` / `jest.config.js` / similar.
- Test functions that don't follow the naming convention the runner expects (`test_*` for pytest, `*.test.js` for jest).
- Skipped tests (`@pytest.mark.skip`, `it.skip(...)`) added "to be enabled later" and never re-enabled.

**Why it matters:** a test that exists but never runs is worse than no test at all, because it creates the illusion of coverage. Verify that the test runner actually executes the new test before considering the code done.
