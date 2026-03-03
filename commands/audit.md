---
description: Run a full Kubernetes security audit of the current repository and write findings to SECURITY-POSTURE.md
---

You are performing a comprehensive Kubernetes security audit of this repository. Follow these steps precisely:

## Step 1: Discover all resources to audit

Search the repository for:
- All Kubernetes manifests (`**/*.yaml`, `**/*.yml`) that contain `kind:` fields
- All Dockerfiles (`**/Dockerfile*`)
- All Helm charts (`**/Chart.yaml`)
- Any CI/CD pipeline files that deploy to Kubernetes (`.github/workflows/*.yml`, etc.)

List each file found before proceeding.

## Step 2: Load all reference files

Load every reference file from this skill:
- `references/secrets-management.md`
- `references/pod-container-security.md`
- `references/network-exposure.md`
- `references/supply-chain-security.md`
- `references/internal-service-auth.md`
- `references/file-handling-security.md`
- `references/llm-ai-security.md`
- `references/helm-manifest-security.md`
- `references/rbac-service-accounts.md`
- `references/observability-incident-response.md`

## Step 3: Audit each file

For each discovered file, check every applicable NEVER/ALWAYS rule. Classify each finding as:

- **CRITICAL** — a NEVER rule is violated (e.g. hardcoded secret, privileged: true, wildcard RBAC)
- **HIGH** — a required ALWAYS control is missing (e.g. no SecurityContext, no resource limits, no NetworkPolicy)
- **MEDIUM** — a best-practice gap that increases risk but is not an immediate violation
- **INFO** — an observation or improvement opportunity

## Step 4: Write SECURITY-POSTURE.md

Create or update `SECURITY-POSTURE.md` in the project root with the full audit results using this structure:

```markdown
# Kubernetes Security Posture

> Last audited: <date> by k8s-security-skills

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

## Step 5: Offer to fix

After writing SECURITY-POSTURE.md, list all CRITICAL and HIGH findings in your reply and ask the user which ones they want fixed immediately. Fix them one file at a time, updating SECURITY-POSTURE.md to reflect resolved findings as you go.
