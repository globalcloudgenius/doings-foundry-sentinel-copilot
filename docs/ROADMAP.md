# Delivery roadmap and definition of done

Status baseline: 2026-10-11. This roadmap distinguishes completed subscription checks from future engineering work.

## Phase 0 — Azure subscription preparation (verified)

- [x] Subscription `Dev` enabled.
- [x] `westus3` recognized by subscription.
- [x] Owner role verified for administrative account.
- [x] Five baseline resource providers registered.
- [x] Terraform available in Cloud Shell.
- [x] USD 5 monthly Cost Management budget configured with four email notifications.
- [x] Candidate GPT-4.1-nano/mini and GPT-4o-mini listed in model catalog.
- [ ] Confirm positive eligible model quota for the chosen deployment type.

## Phase 1 — Repository and delivery security

- [ ] Complete documentation and project structure.
- [ ] Enable GitHub branch protection, required reviewers and dependency scanning.
- [ ] Create federated Azure deployment identity (GitHub Actions OIDC).
- [ ] Assign minimally necessary Azure roles to deployment identity.
- [ ] Provision dedicated protected Terraform state backend, with locking and appropriate permissions.
- [ ] Establish formatting, lint, tests and plan-only PR workflow.

## Phase 2 — Architecture and cost approval

- [ ] Confirm Foundry deployment quota, SKU, prices, processing geography and usage limits.
- [ ] Choose hosting services and validate expected monthly costs.
- [ ] Review Terraform plan, baseline configuration and application threat model.
- [ ] Approve first billable deployment explicitly.

## Phase 3 — Minimum functional product

- [ ] Versioned synthetic incident dataset.
- [ ] Authenticated API with validation, tests and health endpoint.
- [ ] Foundry integration using a working deployment and strict model limits.
- [ ] Evidence-grounded investigation prompt and typed response handling.
- [ ] React UI for browsing incidents and viewing analysis.
- [ ] Secure deployment through reviewed GitHub Actions workflows.

## Phase 4 — Validation and demonstration

- [ ] Unit/integration/security tests.
- [ ] Abuse, authorization, rate-limit, invalid-output, timeout and prompt-injection tests.
- [ ] Verify logging redaction and cost reporting.
- [ ] Document demonstration walkthrough, screenshots and known limitations.
- [ ] Establish teardown and recovery instructions.

## Phase 5 — Optional future work

- [ ] Read-only, least-privilege Sentinel incident connector.
- [ ] More advanced analyst feedback and benchmark evaluation.
- [ ] Optional gated recommendation-to-response workflow (no autonomous execution).

## Release criteria

A phase is complete only when implementation is merged, automated checks pass, and evidence (test results, approved plan, deployment output, or validated demo) is attached. Documentation alone is not proof of a deployed feature.
