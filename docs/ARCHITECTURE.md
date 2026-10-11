# Solution architecture and functional specification

## Architecture diagrams

1. **[Azure solution architecture](diagrams/SOLUTION-ARCHITECTURE.md)** — resource and trust boundaries, GitHub-to-Azure deployment, operational controls.
2. **[Logical investigation flow](diagrams/LOGICAL-INVESTIGATION-FLOW.md)** — end-to-end incident analysis decisions and error paths.

Both are version-controlled diagrams rendered directly by GitHub, and both represent the proposed MVP rather than a currently running deployment.

## Purpose

CloudGenius Azure AI Security Copilot is a planned, human-in-the-loop security-operations demonstration. It combines simulated Microsoft Sentinel incident records with a real Microsoft Foundry-hosted language model to generate analyst-readable findings. It is **not** Microsoft Security Copilot, a replacement for Sentinel, or an autonomous incident-response platform.

**Current state:** Design and documented subscription prerequisites only. Features below are requirements/targets, not statements that software is already built.

## Target workflow

```text
Browser (analyst)
  -> HTTPS web UI (React)
  -> authenticated investigation API (Azure Functions / Python)
  -> server-side incident loader (versioned synthetic JSON fixtures)
  -> schema validation, normalization, redaction and prompt controls
  -> Microsoft Foundry model deployment (approved inference type)
  -> structured output parser + content checks
  -> investigation result displayed with source evidence
```

Initial data is synthetic and stored locally with the application; no live Sentinel workspace, ingestion, connector, or Microsoft 365 E5 license is needed for the initial lab. The UI does not call the model directly. All model requests are initiated server-side.

## End-user functions (planned)

### Incident inventory
- Browse synthetic incident records with incident ID, timestamps, severity, status, affected assets and sample observables.
- Search/filter by severity, technique, account, host, and status.
- Open an incident detail view showing original fields distinctly from generated AI observations.

### Investigation
- Explicit analyst action triggers a model request; **not** continuous real-time detection in the initial release.
- The backend builds a minimized incident summary and sends it to the chosen model.
- Output schema includes executive summary, observed indicators, hypotheses, confidence/uncertainty, recommended next steps and supporting incident-field references.
- Model claims unsupported by input must be labeled inference/uncertain; no fabricated telemetry, log searches, or determinations of compromise.
- Guard against model timeouts, rate limits, malformed JSON, and empty/quota-exhausted responses.

### Security recommendations
- Provide prioritized, analyst-reviewable investigation steps and possible containment guidance.
- Clearly separate evidence, interpretation, and suggested action.
- Never auto-block identities, devices, or network traffic in v1.

### Analyst experience
- Display a loading state, transparent errors, token usage when available, and a history of investigations within the approved retention approach.
- Expose clear demo labels so generated example incidents cannot be mistaken for production customer incidents.

## Implementation components (proposed; subject to cost review)

| Component | Purpose | Selection notes |
|---|---|---|
| React frontend | Incident browser and investigation results | Static hosting option and charges to be evaluated |
| Python Azure Functions API | Authenticated HTTP API, validation, orchestration | Consumption-mode behavior/costs must be checked |
| Microsoft Foundry | Real hosted model inference | Candidate GPT-4.1-nano; deployment quota not yet confirmed |
| Synthetic JSON dataset | Reproducible incident investigation examples | No Sentinel or Log Analytics ingestion charges |
| Managed identity / federated workload auth | Passwordless authentication where supported | Avoid client secrets and API keys in the repo |
| Terraform | Reproducible Azure IaC | Remote state; pinned provider versions |
| GitHub Actions | Validation, review, and approved deployments | Azure OIDC; minimum permissions |
| Cost Management Budget | Cost notifications | Existing USD 5 monthly alert, not a hard stop |

### Region and model processing

Azure resources should be placed in `westus3` where supported. Model inference geography depends on deployment type; **GlobalStandard** and **DataZoneStandard** do not guarantee that processing happens in an Arizona datacenter. Confirm data handling and model deployment type before use. Real production customer telemetry is out of scope for the initial lab.

## API contract (design target)

- `GET /api/health`: liveness only; no secrets or internal Azure diagnostics.
- `GET /api/incidents`: paginated, authorized synthetic incident summaries.
- `GET /api/incidents/{id}`: validated incident detail.
- `POST /api/incidents/{id}/investigate`: user-triggered LLM analysis with stable response schema.
- `GET /api/models/status`: administrative availability and quota-safe status information.

Proposed investigate response fields: `incident_id`, `summary`, `observations[]`, `hypotheses[]`, `recommended_actions[]`, `evidence_refs[]`, `limitations[]`, `model_id`, `generated_at`, `usage`. The actual contract should be defined in a versioned JSON Schema or OpenAPI specification before implementation.

## Future extension: Sentinel

A separate, opt-in integration may read authorized live incident records through Microsoft security APIs, using least-privilege identity, query limiting, pagination, data minimization, audit logging and redaction. Live ingest, ticket updates, automated containment and production SOAR are deliberately **not** included in the minimum-cost MVP.

## Acceptance criteria (future implementation)

1. Fixture records load deterministically; invalid incident input is rejected.
2. A real Foundry model responds successfully under an approved quota.
3. Typed output validates, and user-visible errors are understandable.
4. No model credentials, tokens or customer data appear in frontend bundles, logs or Git history.
5. GitHub PR checks run successfully and deployment approval is enforced.
6. Authentication, authorization, input limits, rate limits and abuse tests pass.
7. Request counts, token use and cost estimates are observable; operations can be halted manually.
8. A demo shows evidence-grounded findings and explicit human approval for decisions.

## Explicit non-goals (MVP)

- Guaranteed real-time Sentinel monitoring or incident response.
- Autonomous remediation, guaranteed detection correctness, or forensic certainty.
- Microsoft 365 E5 integration.
- Production compliance certification, 24/7 availability SLA, or a guaranteed USD 5 hard cost ceiling.
