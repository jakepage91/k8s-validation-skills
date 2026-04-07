---
name: correctness-environment-config
description: Mismatches between code-side environment variable names, Kubernetes Secret/ConfigMap keys, and Helm chart values that cause silent runtime failures.
---

# Environment Configuration and Secret Wiring

> Version: 1.0 | Last Updated: 2026-04-07

These rules catch one of the most common gaps between AI-generated code and a real Kubernetes environment: the code reads from one config name and the cluster provides a different one. The skill can guide the AI to use environment variables, but it cannot tell whether the variable name actually matches what's mounted in the pod. These rules narrow the gap.

---

## Rule 1: Environment Variable Names Must Match Across Code, Manifests, and Helm Values

**ALWAYS** verify that the env var name read in code matches the name set in the Deployment, the key in the Kubernetes Secret or ConfigMap, and any Helm chart values that template it.

Watch for:

- Code reads `os.environ["OPENAI_API_KEY"]` but the Deployment env block sets `OPEN_AI_KEY`.
- The Secret has key `openai-key` but the `secretKeyRef` uses `openai-api-key`.
- Helm values define `openaiKey` but the template references `.Values.openAiKey` (case mismatch).
- The same env var is referenced in two services with two different names.

**ALWAYS** when generating code that reads a new env var, also generate (or explicitly call out) the corresponding manifest/Helm change required.

**Why it matters:** the code starts up, the env var resolves to `None` or `undefined`, and the application either crashes immediately (best case) or silently uses a default (worst case). This kind of bug is invisible to static analysis because both files look correct in isolation. It only shows up when the code runs against the real cluster.

---

## Rule 2: Required Configuration Must Fail Fast

**ALWAYS** validate required configuration at startup and exit with a clear error if anything is missing.

Watch for:

- `api_key = os.environ.get("OPENAI_API_KEY")` followed by code that uses `api_key` without checking it.
- Default values that look reasonable but hide a missing config (`port = int(os.environ.get("PORT", 8080))` is fine; `db_url = os.environ.get("DATABASE_URL", "sqlite:///dev.db")` is dangerous in production).
- Lazy initialization that only fails on the first user request, hours after deploy.

**ALWAYS** prefer:

```python
api_key = os.environ.get("OPENAI_API_KEY")
if not api_key:
    raise RuntimeError("OPENAI_API_KEY is required")
```

**Why it matters:** a service that starts up successfully but is missing critical config will pass health checks, get traffic, and start failing in user-visible ways. Failing fast at startup turns a silent runtime bug into an obvious deploy-time error.

---

## Rule 3: ConfigMap and Secret Updates Must Be Reflected in Pod Restarts

**ALWAYS** assume that env vars sourced from a ConfigMap or Secret are read at pod start. Updating the ConfigMap does not update the running pod.

Watch for:

- Documentation or runbooks that say "edit the ConfigMap to change the value" without mentioning a rollout.
- Code that re-reads an env var on every request expecting it to change (env vars are immutable for the process lifetime).
- Hot-reload patterns that don't actually trigger when the underlying ConfigMap changes.

**Why it matters:** teams update a ConfigMap, expect the change to take effect, and then spend hours debugging why the new value isn't being used. Either trigger a rollout (`kubectl rollout restart deploy/...`) or use a config-reload sidecar. Don't pretend env vars are dynamic.
