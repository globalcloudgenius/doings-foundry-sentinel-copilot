# Security, operational controls and cost governance

## Trust boundaries

Browser inputs and incident text are untrusted. The investigation service authenticates the caller, authorizes access to an incident, validates a strict schema, minimizes data, and only then constructs an LLM request. Treat model output as untrusted suggestions, not privileged commands.

## Secure engineering controls (planned)

- **Identity:** Prefer Entra-backed user authentication and server-side managed identity where supported. Assign workload-specific RBAC. GitHub Actions uses OIDC federation rather than a client secret.
- **Permissions:** Separate infrastructure deployment identity from runtime identity. Scope permissions to specific RGs, services, storage containers and actions. No GitHub subscription Owner.
- **Network and HTTPS:** Require TLS, validate CORS origins, and avoid public administrative interfaces. Private networking may be deferred if its cost exceeds the lab budget; document any residual exposure.
- **Input security:** Verify incident IDs, size limits, JSON schemas and content types; defend against prompt injection embedded in incident descriptions and logs.
- **Output safety:** Use structured response validation, escape displayed content, and require human review. Never execute model-generated commands automatically.
- **Data:** Synthetic fixtures only at first. Exclude real credentials, PII, tokens and secrets. Review logging/retention before real data.
- **Secrets:** Avoid storing secrets in GitHub or Terraform state. Where no managed-identity option exists, use a protected secret store and rotate secrets.
- **CI/CD:** Protect main, require PR review, lint and scan IaC and dependencies, pin dependencies/actions, and use GitHub environments with deployment approval.
- **Observability:** Minimal structured telemetry, correlation IDs and redacted errors. Disable unnecessary high-volume logging.
- **Incident response:** Document disabling public access, stopping Functions, revoking OIDC trust/roles and removing model deployments.

## USD 5 monthly budget: what it does and does not do

Existing Azure Cost Management configuration: `budget-dev-monthly-5usd` for USD 5 per month; actual-spend notifications to `isaacobaro01@gmail.com` at USD 3, 4, 4.50 and 5. These settings were confirmed in the Azure API response on 2026-10-11.

- A budget **does not** terminate running resources or block billing.
- Cost data and alerts can arrive late.
- Static hosting, storage, logs, Functions and inference may each incur charges depending on service tiers and consumption.
- Quota (TPM/RPM) is not the same thing as a monthly spending cap.
- Never enable provisioned model throughput or unreviewed always-on services under a tiny experimental budget.

## Recommended lab operating policy

1. Use one dev environment and synthetic incident fixtures.
2. Prefer low-utilization, consumption-based services after checking current regional price lists and any free allowances.
3. Set server-side max input/output tokens, request size, rate limiting, per-user and overall daily invocation caps, and a kill switch.
4. Return an explicit quota/budget-unavailable response rather than falling back silently to a more expensive model.
5. Confirm per-request cost estimate and track usage through minimal telemetry.
6. Review cost daily during deployment/testing and delete idle resources promptly.
7. Require explicit approval for any change that could create recurring cost.
8. Prefer manual start/stop of the lab; do not promise spend stops at a precise dollar amount.

## Deployment gates

**No automatic production-like apply from unreviewed pull requests.** Use `terraform fmt`, `terraform validate`, policy/security checks and a reviewed `terraform plan`; require protected-environment approval to apply. Terraform state must be remote, access-restricted and excluded from Git.

## Disclosure

This repository documents an enterprise-inspired **learning project**, not a certified production security service. Model-generated findings are advisory. Human analysts remain responsible for validation and response.
