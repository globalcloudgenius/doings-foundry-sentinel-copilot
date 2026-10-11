# Environment prerequisites and verification record

**Project:** CloudGenius Azure AI Security Copilot — Real-Time Security Incident Investigation with Microsoft Foundry  
**Evidence date:** 2026-10-11  
**Environment:** Development / lab  
**Evidence type:** Azure CLI output supplied by the project owner. This document is a point-in-time record, not continuous monitoring.

## Verified

| Area | Result | Evidence / limitation |
|---|---|---|
| Subscription | `Dev`, Enabled | Azure CLI `az account show` |
| Primary Azure region | `westus3` (West US 3) | `az account list-locations` returned region |
| Administrative account | `labadmin@doings.party` | Azure CLI signed-in account |
| Subscription RBAC | Owner (direct subscription role) | `az role assignment list` showed Owner |
| Terraform on inspection shell | v1.16.4 | `terraform version` |
| Azure CLI authentication | Working | Read operations completed successfully |
| Existing resources | No resources returned | `az resource list` produced an empty table at inspection time |
| Budget | `budget-dev-monthly-5usd`, USD 5 monthly | Successful Azure Cost Management budget PUT response |
| Alert thresholds | 60%, 80%, 90%, 100% actual cost | Enabled in returned budget resource |
| Alert email | `isaacobaro01@gmail.com` | Present in the budget's `contactEmails` |
| Budget start | 2026-10-01 UTC | Returned budget definition |
| Cognitive Services provider | Registered | `az provider show` |
| Web provider | Registered | `az provider show` |
| Storage provider | Registered | `az provider show` |
| Managed Identity provider | Registered | `az provider show` |
| Insights provider | Registered | `az provider show` |
| Model catalog | `gpt-4.1-nano`, `gpt-4.1-mini`, `gpt-4o-mini` returned for `westus3` | `az cognitiveservices model list` |

## Not yet verified / incomplete

- **Actual Foundry deployment eligibility:** Model catalog listings are not deployment approvals. GPT-4o-mini Standard-related usage entries were reported at zero for the checked categories; applicable GPT-4.1-nano/mini online deployment quota was not demonstrated.
- **GitHub-to-Azure federation:** No OIDC federated credential or deployment identity has yet been verified.
- **Least-privilege deployment RBAC:** Administrator Owner permissions do not mean GitHub should have Owner.
- **Terraform remote state:** No backend storage, locking, or RBAC has been verified.
- **App infrastructure:** No Functions app, Foundry resource/project, model deployment, or live API has been verified.
- **Email delivery:** Budget alert rules were created, but actual email delivery was not tested.
- **Capacity and pricing:** Region availability does not ensure SKU capacity or a deployment within USD 5. Confirm current prices before apply.
- **Additional resource providers:** Register further providers only when Terraform's reviewed configuration requires them.

## Known environment identifiers

- Subscription display name: `Dev`
- Subscription ID: `7f54ad46-7f2f-453e-92dd-69f881e38371`
- Entra tenant ID: `180b5be7-2cfa-4ad1-8b24-11a384cc04f7`
- Preferred location: `westus3`
- Naming convention suffix: `dev-wus3`

These identifiers are metadata, not passwords; nevertheless, avoid copying personal identifiers into logs or demo screenshots unnecessarily.

## Verification commands (read-only)

```bash
az account set --subscription "Dev"
az account show --query '{Name:name,State:state,Tenant:tenantId}' -o json
az account list-locations --query "[?name=='westus3'].{Name:name,Display:displayName}" -o table
for p in Microsoft.CognitiveServices Microsoft.Web Microsoft.Storage Microsoft.ManagedIdentity Microsoft.Insights; do
  az provider show --namespace "$p" --query '{Provider:namespace,Status:registrationState}' -o table
done
az cognitiveservices model list --location westus3 --query "[?model.name=='gpt-4.1-nano' || model.name=='gpt-4.1-mini' || model.name=='gpt-4o-mini'].model.{name:name,version:version}" -o table
```

## Pre-deployment gates

1. Confirm repository ownership, main branch protections, and appropriate GitHub Environment approvals.
2. Create a dedicated federated identity. Limit its role assignments to explicitly approved resource groups and narrowly scoped operations.
3. Initialize a private Terraform state backend with secured access; never commit `.tfstate`, `.tfvars` containing secrets, or credentials.
4. Confirm selected model, version, deployment type, available quota, pricing, and data-processing geography.
5. Review a Terraform plan and monthly cost forecast before applying.
6. Implement application-level request/token budgets, abuse controls, and a manual kill switch.
7. Deploy only after an explicit approval. Perform negative security tests before exposing the application.

> **Budget warning:** Azure Budgets are notifications, not enforcement. Actual costs can exceed USD 5 because of reporting delays, metering, or unbounded usage.
