---
name: densify-rightsize-containers
description: >-
  Pull Kubex (formerly Densify) container right-sizing recommendations for a Kubernetes cluster —
  recommended CPU/memory requests and limits, OOM-kill history and uptime prediction — so an agent
  can propose manifest changes.
api: Densify / Kubex Container Optimization API
generated: '2026-09-06'
method: generated
source: openapi/densify-kubernetes-*.yaml, openapi/densify-authorize-openapi.yaml, conventions/densify-conventions.yml
operations:
  - authorize-user
  - listKubernetesClusters
  - getKubernetesClusterDetails
  - getKubernetesClusterContainers
  - getKubernetesClusterContainersDetailed
  - ping
---

# Read container right-sizing recommendations

## Preconditions

- Base URL `https://<instance>.kubex.ai/api/v2`; health check is one level up at
  `https://<instance>.kubex.ai/api/ping`.
- The cluster must already be forwarding data. Kubex collects container telemetry through the
  in-cluster collection stack (`helm repo add kubex https://densify-dev.github.io/helm-charts`,
  then the `kubex-collection-stack` / `kubex-automation-stack` chart). An empty cluster list means
  collection is not running, not that the API failed.

## 1. Check reachability (optional, unauthenticated)

`ping` — `GET /api/ping`. Takes an optional `timeout` query parameter, integer seconds, range
15–180, default 30. This is the only operation in the whole API that needs no token.

## 2. Authenticate

`authorize-user` — `POST /authorize`. Bearer JWT, 60-minute lifetime. See
`authentication/densify-authentication.yml`.

## 3. List the clusters

`listKubernetesClusters` — `GET /kubernetes/clusters`. Returns every cluster Kubex is collecting
from, unfiltered. The natural key is `clusterName`, not an opaque id.

## 4. Inspect one cluster

`getKubernetesClusterDetails` — `GET /kubernetes/clusters/{clusterName}`. Returns the
namespace → pod → container tree. Use this to scope which workloads you are about to touch.

## 5. Get the recommendations

`getKubernetesClusterContainers` — `GET /kubernetes/clusters/{clusterName}/containers`
returns the base recommendation set for every container manifest in the cluster.

`getKubernetesClusterContainersDetailed` — the same path with **`details=true`** — adds
`predictedUptime`, `configLastChangedOn`, `nodeGroup`, `oomKills_last7days`, `dateFirstAudited`
and `dateLastAudited`.

Prefer the detailed form when you are about to propose a change: `oomKills_last7days` and
`predictedUptime` are what tell you whether a memory reduction is safe, and
`configLastChangedOn` tells you whether someone has already edited the manifest since the
recommendation was computed.

Two container attributes govern automation eligibility and are worth reading before proposing
anything:

- **Kubex Automation** — whether the container is eligible for automation. Set once, static, not
  inherited from cluster, node group or any higher level.
- **Kubex Policy** — which policy applies when automation is enabled.

## 6. Applying the change

This REST API is **read-only with respect to your cluster**. Nothing here mutates a workload.
Changes are applied either by you (editing manifests / IaC) or by the in-cluster Kubex Automation
Engine under its own `AutomationStrategy` / `ProactivePolicy` / `StaticPolicy` CRDs and safety
guardrails. Do not tell a user that calling this API resized anything.

## Gotchas

- **No pagination.** A large cluster returns one large array. Budget for it.
- **Omitted fields.** "Fields with no value may be omitted from the response" — absence is normal.
- **No dry-run.** There is no preview mode; the GET *is* the preview.
- **No rate-limit headers.** Nothing tells you how much budget is left. The only 429 published is
  on `/authorize`.
