---
description: Run a Kubernetes security audit and write findings to SECURITY-POSTURE.md. Optionally scope to a specific path: /k8s-security:audit llm-gateway/
---

You are performing a comprehensive Kubernetes security audit. Follow these steps precisely:

## Step 0: Determine scope

Check whether a path argument was provided (e.g. `/k8s-security:audit llm-gateway/`).

- **If a path was provided**: restrict all discovery and auditing to that directory. Note the scope at the top of SECURITY-POSTURE.md: `> Scope: <path>`.
- **If no path was provided**: audit the entire repository from the root.

## Step 1: Discover all resources to audit

Within the determined scope, search for:
- All Kubernetes manifests (`**/*.yaml`, `**/*.yml`) that contain `kind:` fields
- All Dockerfiles (`**/Dockerfile*`)
- All Helm charts (`**/Chart.yaml`)
- Any CI/CD pipeline files that deploy to Kubernetes (`.github/workflows/*.yml`, etc.)

List each file found before proceeding.

## Step 2: Load only the relevant reference files

Based on what was found in Step 1, load only the reference files that apply. Do **not** load references for artifact types that were not discovered.

| Condition | Load |
|---|---|
| Any k8s manifest found | `references/secrets-management.md` |
| Any manifest with `kind: Deployment`, `Pod`, `DaemonSet`, `StatefulSet`, `Job`, or `CronJob` | `references/pod-container-security.md` |
| Any manifest with `kind: Service`, `Ingress`, or `NetworkPolicy` | `references/network-exposure.md` |
| Any manifest with `kind: Role`, `ClusterRole`, `RoleBinding`, or `ClusterRoleBinding`; or any `serviceAccountName` reference | `references/rbac-service-accounts.md` |
| Any `Dockerfile*` or CI/CD pipeline file found | `references/supply-chain-security.md` |
| Any `Chart.yaml` (Helm chart) found | `references/helm-manifest-security.md` |
| Any manifest or code file referencing inter-service auth, mTLS, JWT, or service mesh annotations | `references/internal-service-auth.md` |
| Any application code file (non-manifest) found | `references/file-handling-security.md` |
| Any file with LLM/AI indicators: filenames or content containing `llm`, `openai`, `anthropic`, `langchain`, `embeddings`, `prompt`, `completion` | `references/llm-ai-security.md` |
| Any k8s manifest or application code found (i.e. almost always) | `references/observability-incident-response.md` |

List which reference files were loaded and which were skipped (with the reason) before proceeding.

## Step 3: Audit each file

For each discovered file, check every applicable NEVER/ALWAYS rule. Classify each finding as:

- **CRITICAL** — a NEVER rule is violated (e.g. hardcoded secret, privileged: true, wildcard RBAC)
- **HIGH** — a required ALWAYS control is missing (e.g. no SecurityContext, no resource limits, no NetworkPolicy)
- **MEDIUM** — a best-practice gap that increases risk but is not an immediate violation
- **INFO** — an observation or improvement opportunity

## Step 4: Write SECURITY-POSTURE.md

Create or update `SECURITY-POSTURE.md` in the project root with the full audit results using this structure. Then ensure `SECURITY-POSTURE.md` is listed in `.gitignore` — append the entry if missing, creating `.gitignore` if it does not exist.

```markdown
# Kubernetes Security Posture

> Last audited: <date> by k8s-security-skills
> Scope: <path or "entire repository">

## Summary

| Severity | Count |
|----------|-------|
| CRITICAL | N |
| HIGH     | N |
| MEDIUM   | N |
| INFO     | N |

---

## Findings by File

### `<path/to/file.yaml>`

#### CRITICAL: <Finding title>
- **Rule**: NEVER rule reference (e.g. secrets-management Rule 1)
- **Location**: line number or field path
- **Risk**: What an attacker could do if this is not fixed
- **Fix**: Exact remediation with corrected YAML/code snippet

#### HIGH: <Finding title>
...

---

## Files with no findings

- `path/to/clean-file.yaml` — all controls present
```

## Step 5: Summarise and recommend

After writing SECURITY-POSTURE.md, output a concise summary in the chat:

- The total finding counts by severity
- The top 3 most urgent issues with a one-line description of each
- A reminder that **no files have been modified** — this audit is read-only

Do NOT auto-fix anything. The manifests may be running in production and changes require a deliberate deployment process. Instead, for each CRITICAL and HIGH finding, include a concrete recommendation in SECURITY-POSTURE.md under a `**Recommended fix:**` field so the user has everything they need to act when ready.

If the user explicitly asks to fix a specific finding after the audit, address it then — one file at a time, with the user's confirmation before writing.
