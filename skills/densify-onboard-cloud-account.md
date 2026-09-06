---
name: densify-onboard-cloud-account
description: >-
  Onboard an AWS account, Azure subscription or GCP project into Kubex (formerly Densify) for
  optimization analysis, then retrieve the resulting right-sizing recommendations — as JSON or
  as a Terraform map for infrastructure-as-code.
api: Densify / Kubex Public Cloud API
generated: '2026-09-06'
method: generated
source: openapi/densify-public-cloud-*.yaml, conventions/densify-conventions.yml, errors/densify-problem-types.yml
operations:
  - authorize-user
  - analyzeAws
  - listAwsAnalyses
  - getAwsRecommendations
  - analyzeAzure
  - listAzureAnalyses
  - getAzureAnalysisStatus
  - listAzureRecommendations
  - analyzeGcp
  - listGcpAnalyses
  - listGcpRecommendations
  - deleteAwsAnalysis
  - deleteAzureAnalysisAudit
  - deleteGcpAnalysisAudit
---

# Onboard a cloud account and read its recommendations

## Before you start

- The base URL is **your own tenant host**: `https://<instance>.kubex.ai/api/v2` (or
  `https://<instance>.densify.com/api/v2`). There is no shared API hostname — `api.densify.com`
  is the marketing site, not an API.
- You need credentials for an **active AND API-enabled** Kubex user. An active user that is not
  API-enabled receives a token that every downstream call rejects.
- Recommendations only exist after data collection and analysis have run. Collection runs
  nightly; the first result set is not immediate.

## 1. Get a token

`POST /authorize` (`authorize-user`) with `{ "userName": ..., "pwd": ... }`.

Read `apiToken` from the response and send it as `Authorization: Bearer <apiToken>` on every
other call. The token lasts **60 minutes** — long enough for a pipeline run, so do not
re-authorize per request. On repeated credential failure the endpoint returns **429** with a
progressive delay; slow down rather than retrying tighter.

## 2. Onboard the account

| Cloud | Operation | Path |
|---|---|---|
| AWS | `analyzeAws` | `POST /analysis/cloud/aws/analyze` |
| Azure | `analyzeAzure` | `POST /analysis/azure/analyze` |
| GCP | `analyzeGcp` | `POST /analysis/gcp/analyze` |

Supply the account/subscription/project plus its credential (IAM role for AWS, service principal
for Azure, service account for GCP). Optionally supply a `webHook` to have results delivered.

**This call is not idempotent in the replay sense.** Kubex has no `Idempotency-Key` header on any
operation. Re-submitting the same account does not create a duplicate — it re-runs analysis — but
re-submitting it with a *different* `webHook` silently rebinds the webhook. Decide the webhook
once.

For Azure and GCP a historical audit can be requested instead (`triggerAdhocAudit=true` with
`startDayOffset` / `endDayOffset` on Azure; `historicalAuditGcp` on GCP). GCP rejects the request
if collection is already in progress.

## 3. Find the analysisId

`listAwsAnalyses` / `listAzureAnalyses` / `listGcpAnalyses` return the analyses with the
`analysisId` every later call needs. For Azure you can also poll
`getAzureAnalysisStatus` (`GET /analysis/azure/{subscriptionId}/status`).

## 4. Read the recommendations

| Cloud | Operation | Path |
|---|---|---|
| AWS | `getAwsRecommendations` | `GET /analysis/cloud/aws/{analysisId}/results` |
| Azure | `listAzureRecommendations` | `GET /analysis/cloud/azure/{analysisId}/results` |
| GCP | `listGcpRecommendations` | `GET /analysis/cloud/gcp/{analysisId}/results` |

Choose the representation with the **`Accept` header**, not a different URL:

- `application/json` — an array of recommendation objects.
- `application/terraform-map` — a Terraform-style map keyed by each system's `provisioningId`,
  for feeding straight into IaC.

Filter with flat query pairs: `recommendationType=Upsize`, `region`, `serviceType`, `entityId`,
`accountIdRef`, `currentType`, `recommendedType`, `dataQuality`. Add `includeAttributes=true` to
get the `attributes` block.

**There is no pagination.** No `limit`, `offset`, `page` or cursor exists on these endpoints and
no next-link is returned — the full array comes back. Narrow with the filters above rather than
expecting to page.

Fields with no value may be omitted entirely from a response object. Treat absence as "no value",
not as an error.

## 5. Offboarding

`deleteAwsAnalysis` / `deleteAzureAnalysisAudit` / `deleteGcpAnalysisAudit`
(`DELETE /analysis/cloud/{aws|azure|gcp}/{analysisId}`) delete the **Cloud Connection and the
Analysis** together. There is no undelete and **no restore window is published** — treat this as
permanent and re-onboard from scratch if you need it back.

## Error handling

Errors are plain JSON `{ "message": ..., "status": ... }` — **not** RFC 9457 problem+json.

| Status | Meaning | What to do |
|---|---|---|
| 400 | invalid parameters, or a conflicting in-progress operation | fix and retry; do not retry unchanged |
| 401 | token missing, invalid or expired | re-run `authorize-user` and retry once |
| 403 | trial or subscription expired | stop; this is an entitlement problem |
| 404 | analysis not found | re-list analyses to get a current `analysisId` |
| 429 | authorization throttled (progressive delay) | back off |
| 500 | server or connectivity error | retry with backoff |
