---
name: Kubex
description: Use when optimizing Kubernetes container and node resources, configuring automated rightsizing, analyzing cloud infrastructure costs, or retrieving optimization recommendations via API. Kubex is for SREs, platform engineers, and FinOps teams managing Kubernetes and multi-cloud infrastructure.
metadata:
    mintlify-proj: kubex
    version: "1.0"
---

# Kubex Skill Reference

## Product Summary

Kubex is an autonomous resource optimization platform for Kubernetes and cloud infrastructure. It combines machine learning analysis, policy-driven recommendations, and controlled automation to reduce infrastructure waste while maintaining workload stability. Agents use Kubex to rightsize containers and nodes, optimize GPU allocation, analyze cloud costs, and apply recommendations safely at scale.

**Key files and commands:**
- Helm chart: `kubex/kubex-automation-engine` (primary automation component)
- Data collection: `kubex/kubex-collection-stack` (Helm chart for Kubernetes telemetry)
- API base: `https://<instance>.kubex.ai/api`
- Config: Helm `values-edit.yaml` for automation engine; CRDs for policy/strategy
- CLI: `kubectl` for managing automation resources; `helm` for deployment

**Primary docs:** https://docs.kubex.ai

## When to Use

Reach for this skill when:

- **Container optimization:** Analyzing CPU/memory waste, rightsizing requests/limits, or applying recommendations to workloads
- **Automation setup:** Installing Kubex Automation Engine, configuring policies, or managing strategy/policy CRDs
- **Cloud cost analysis:** Onboarding AWS/Azure/GCP accounts, retrieving instance recommendations, or analyzing scale group optimization
- **API integration:** Retrieving cluster details, container recommendations, or cloud analysis results programmatically
- **Troubleshooting:** Diagnosing automation failures, verifying data collection, or checking safety control behavior
- **Policy management:** Creating namespace or cluster-scoped automation strategies, proactive policies, or static policies
- **Data collection:** Connecting Kubernetes clusters via Helm, configuring multi-cluster setups, or using observability platforms

## Quick Reference

### Helm Installation Commands

| Task | Command |
|------|---------|
| Add Kubex repo | `helm repo add kubex https://densify-dev.github.io/helm-charts && helm repo update` |
| Install CRDs | `helm install kubex-crds kubex/kubex-crds --namespace kubex --create-namespace` |
| Install Automation Engine | `helm install kubex-automation-engine kubex/kubex-automation-engine --namespace kubex -f values-edit.yaml` |
| Upgrade after config change | `helm upgrade kubex-automation-engine kubex/kubex-automation-engine --namespace kubex -f values-edit.yaml` |
| Install data collector | `helm install kubex-collection kubex/kubex-collection-stack --namespace kubex -f collector-values.yaml` |

### Required Automation Engine Values

```yaml
kubex:
  url:
    host: your-instance.kubex.ai
    scheme: https
  clusterName: my-cluster

kubexCredentials:
  username: your-username
  epassword: your-encrypted-password  # or use createSecrets: false with external secret
```

### Key CRD Types

| CRD | Scope | Purpose |
|-----|-------|---------|
| `AutomationStrategy` | Namespaced | Define how resource changes are allowed (CPU/memory enablement, resize behavior) |
| `ClusterAutomationStrategy` | Cluster | Cluster-wide strategy for resource changes |
| `ProactivePolicy` | Namespaced | Apply recommendation-driven automation to specific workloads |
| `ClusterProactivePolicy` | Cluster | Cluster-wide recommendation-driven policy |
| `StaticPolicy` | Namespaced | Apply fixed resource values when recommendations unavailable |
| `GlobalConfiguration` | Cluster | Cluster-wide runtime behavior (refresh intervals, automation on/off, safety thresholds) |

### API Endpoints (REST)

| Endpoint | Purpose |
|----------|---------|
| `GET /kubernetes/clusters` | List all connected clusters |
| `GET /kubernetes/clusters/{clusterName}` | Get cluster details (namespaces, pods, containers) |
| `GET /kubernetes/clusters/{clusterName}/containers` | List container recommendations for cluster |
| `GET /kubernetes/clusters/{clusterName}/containers?details=true` | Get detailed container metrics (OOM kills, uptime, audit history) |
| `POST /analysis/cloud/aws/analyze` | Onboard AWS account for analysis |
| `GET /analysis/cloud/aws/{analysisId}/results` | Get AWS recommendations |
| `POST /authorize` | Obtain API token (required for all requests) |

### Diagnostic Commands

```bash
# Verify automation engine deployment
kubectl get deployment kubex-automation-engine -n kubex
kubectl get pods -n kubex -l control-plane=controller-manager

# Check policy and strategy resources
kubectl get clusterproactivepolicy,proactivepolicy,clusterstaticpolicy,staticpolicy -A

# View global configuration
kubectl get globalconfiguration global-config -o yaml

# Check for precheck failures
kubectl get events -A --field-selector reason=PrecheckFailed

# View recent rightsizing activity
kubectl logs -n kubex -l control-plane=controller-manager -c manager --since=10m | grep 'rightsizing summary'
```

## Decision Guidance

### When to Use Helm-Managed vs Manual CRs

| Scenario | Use Helm-Managed | Use Manual CRs |
|----------|------------------|----------------|
| Baseline cluster-wide policy | ✓ | |
| Namespace-level exceptions | | ✓ |
| GitOps-driven policy management | | ✓ |
| Simple single-policy setup | ✓ | |
| Complex multi-team policies | | ✓ |
| Rapid iteration during testing | ✓ | |

**Pattern:** Use Helm for baseline (`scope` and `policy.policies` in values), then layer manual CRs for exceptions.

### When to Use In-Place Resize vs Eviction

| Condition | In-Place Resize | Eviction Fallback |
|-----------|-----------------|-------------------|
| Kubernetes 1.33+ | ✓ (preferred) | |
| Older Kubernetes | | ✓ (automatic) |
| Stateless workloads | ✓ | ✓ |
| StatefulSets with PVCs | ✓ (safer) | ✓ |
| DaemonSets | | ✓ (required) |

**Default:** Engine attempts in-place resize first; falls back to eviction if policy or cluster version requires it.

### Container vs Node Optimization

| Goal | Use Container Optimization | Use Node Optimization |
|------|---------------------------|----------------------|
| Reduce CPU/memory waste | ✓ | |
| Improve scheduler packing | ✓ | |
| Right-size cloud instances | | ✓ |
| Optimize scale groups | | ✓ |
| Reduce OOM kills | ✓ | |
| Eliminate idle nodes | | ✓ |

## Workflow

### 1. Connect a Kubernetes Cluster

1. **Prepare Helm values** for data collection (cluster name, Kubex instance URL)
2. **Deploy data collector** using `kubex-collection-stack` Helm chart
3. **Verify data flow** in Kubex UI (Connections tab shows status within 1 hour)
4. **Wait for analysis** (overnight batch processing; results available next morning)

### 2. Set Up Automation Engine

1. **Prepare values file** (`values-edit.yaml`) with Kubex credentials and cluster name
2. **Install CRDs** via `helm install kubex-crds`
3. **Install Automation Engine** via `helm install kubex-automation-engine`
4. **Verify deployment** using diagnostic commands above
5. **Create strategy and policy** (Helm-managed or manual CRs)
6. **Monitor logs** for policy resolution and recommendation ingestion

### 3. Retrieve Container Recommendations via API

1. **Authenticate** with `POST /authorize` to get token
2. **List clusters** with `GET /kubernetes/clusters`
3. **Get container recommendations** with `GET /kubernetes/clusters/{clusterName}/containers`
4. **Parse response** for CPU/memory/storage/GPU recommendations
5. **Apply recommendations** manually or via Automation Engine

### 4. Analyze Cloud Infrastructure

1. **Onboard cloud account** with `POST /analysis/cloud/{provider}/analyze` (AWS/Azure/GCP)
2. **Poll analysis status** until complete
3. **Retrieve recommendations** with `GET /analysis/cloud/{provider}/{analysisId}/results`
4. **Review optimization types** (Downsize, Upsize, Terminate, Modernize)
5. **Download impact reports** for stakeholder review

### 5. Troubleshoot Automation Issues

1. **Check deployment health** (`kubectl get deployment kubex-automation-engine`)
2. **Verify policy resolution** in controller logs
3. **Look for precheck failures** (`kubectl get events --field-selector reason=PrecheckFailed`)
4. **Confirm GlobalConfiguration** exists and automation is enabled
5. **Review safety controls** (HPA/VPA compatibility, LimitRange, ResourceQuota, node headroom)
6. **Check webhook health** in GlobalConfiguration status

## Common Gotchas

- **Data collection delay:** Kubex performs overnight batch analysis. Newly connected clusters show results the next morning, not immediately.
- **Missing cluster name:** `kubex.clusterName` in Helm values must match the cluster identifier sent by data collector. Mismatch causes recommendations to not appear.
- **Credentials not encrypted:** If using `createSecrets: true`, `kubexCredentials.epassword` must be encrypted. Use Kubex UI to generate encrypted password.
- **Policy not applied:** Verify `ProactivePolicy` references correct `AutomationStrategy` by name and that namespace/label selectors match target workloads.
- **Automation disabled silently:** Check `globalConfiguration.automationEnabled` is `true`. If false, no mutations occur even if policies are correct.
- **Webhook not ready:** Mutations fail if webhook is unhealthy. Check webhook pod logs and `GlobalConfiguration.webhookHealth` status.
- **Protected namespaces:** `globalConfiguration.protectedNamespacePatterns` may exclude system namespaces (e.g., `kube-*`). Verify patterns don't accidentally block intended targets.
- **HPA/VPA conflicts:** Automation Engine skips workloads with active HPA or VPA unless policy explicitly allows. Check logs for `HPA/VPA compatibility` messages.
- **Stale recommendations:** API returns recommendations with `analyzedOn` timestamp. Automation only applies fresh recommendations; old data is ignored.
- **API token expiration:** Tokens are temporary and must be refreshed daily. Reauthorize if API calls fail with 401.
- **Deprecated Automation Controller:** Old `kubex-automation-controller` chart is deprecated. Use `kubex-automation-engine` instead.

## Verification Checklist

Before submitting work:

- [ ] Data collection deployed and showing "Connected" status in Kubex UI
- [ ] Automation Engine pods running: `kubectl get pods -n kubex -l control-plane=controller-manager`
- [ ] CRDs installed: `kubectl get crd | grep kubex`
- [ ] GlobalConfiguration exists: `kubectl get globalconfiguration global-config`
- [ ] Strategy and policy CRs created and no validation errors
- [ ] No repeated `PrecheckFailed` events for target workloads
- [ ] Recent `rightsizing summary` logs present (within last 10 minutes)
- [ ] Webhook validating/mutating configurations present: `kubectl get validatingwebhookconfigurations,mutatingwebhookconfigurations | grep kubex`
- [ ] API authentication works: `POST /authorize` returns token
- [ ] Container recommendations visible in API: `GET /kubernetes/clusters/{clusterName}/containers` returns data
- [ ] Cloud analysis complete: `GET /analysis/cloud/{provider}/{analysisId}/results` returns recommendations

## Resources

**Comprehensive navigation:** https://docs.kubex.ai/llms.txt

**Critical documentation pages:**
- [Kubex Overview](https://docs.kubex.ai/docs-kubex/Content/General/Kubex_Overview) — Core capabilities and use cases
- [Automation Engine Getting Started](https://docs.kubex.ai/docs-kubex/Content/Kubex/Automation_Engine_Getting_Started) — Step-by-step installation and configuration
- [API Introduction](https://docs.kubex.ai/docs-api/WebHelp_Densify_API_Cloud/Content/API_Guide/Introduction) — REST API overview and endpoints

---

> For additional documentation and navigation, see: https://docs.kubex.ai/llms.txt