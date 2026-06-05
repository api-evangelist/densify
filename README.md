# Densify (densify)

Densify (now Kubex) provides a machine learning powered cloud and container optimization platform that continuously right-sizes resources to reduce cost and improve performance across Kubernetes, public cloud, and virtualized environments. The Densify REST API exposes optimization analysis, recommendations, account and cluster inventory, and systems data so that optimization can be embedded into CI/CD pipelines, infrastructure as code templates, and FinOps workflows.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/densify/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/densify/refs/heads/main/apis.yml)

## Scope

- **Type:** Index
- **Position:** Producer
- **Access:** 3rd-Party

## Tags

- Cloud Cost
- Container Optimization
- FinOps
- Kubernetes
- Machine Learning
- Recommendations
- Right-Sizing

## Timestamps

- **Created:** 2026-03-16
- **Modified:** 2026-04-28

## APIs

### Densify Public Cloud API

The Densify Public Cloud REST API exposes optimization analysis, recommendations, account and instance inventory, and systems data for AWS, Azure, and Google Cloud environments. The API uses JSON over HTTPS, follows a resource-oriented design under the /api/v2/ path, and authenticates with JWT bearer tokens obtained from the /authorize endpoint. Common use cases include integrating right-sizing recommendations into Terraform, CloudFormation, and CI/CD pipelines.

- **Human URL:** [https://www.densify.com/docs-api/WebHelp_Densify_API_Cloud/Content/API_Guide/Introduction.htm](https://www.densify.com/docs-api/WebHelp_Densify_API_Cloud/Content/API_Guide/Introduction.htm)
- **Base URL:** `https://api.densify.com/api/v2`

#### Tags

- Cloud
- FinOps
- Optimization
- Recommendations
- Right-Sizing

#### Properties

- [Documentation](https://www.densify.com/docs-api/WebHelp_Densify_API_Cloud/Content/API_Guide/Introduction.htm)
- [Reference](https://www.densify.com/docs/WebHelp_Densify_Cloud/Content/Resources/PDFs/Densify-API-Reference-Guide.pdf)
- [Postman](https://www.densify.com/dev) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Postman Collection](collections/densify.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/densify.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Densify Container Optimization API

The Densify Container Optimization REST API provides programmatic access to container right-sizing recommendations across Amazon EKS, AKS, GKE, OpenShift, and self-managed Kubernetes footprints. It exposes endpoints for clusters, namespaces, controllers, container groups, and recommended requests/limits for CPU and memory based on machine learned utilization patterns.

- **Human URL:** [https://www.densify.com/docs-api/WebHelp_Densify_API/Content/Densify_API.htm](https://www.densify.com/docs-api/WebHelp_Densify_API/Content/Densify_API.htm)
- **Base URL:** `https://api.densify.com/api/v2`

#### Tags

- Containers
- Kubernetes
- Optimization
- Recommendations
- Right-Sizing

#### Properties

- [Documentation](https://www.densify.com/docs-api/WebHelp_Densify_API/Content/Densify_API.htm)
- [Developer  Resources](https://www.densify.com/dev)
- [Postman Collection](collections/densify.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/densify.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [LinkedIn](https://www.linkedin.com/company/densify)
- [Website](https://www.densify.com)
- [Portal](https://portal.densify.com/)
- [Documentation](https://docs.densify.com)
- [Developer  Resources](https://www.densify.com/dev)
- [Documentation  Landing](https://portal.densify.com/docs-landing/)
- [Resources](https://www.densify.com/resources)
- [Blog](https://www.densify.com/blog)
- [Pricing](https://www.densify.com/pricing)
- [Free  Trial](https://www.densify.com/free-trial)
- [Support](https://www.densify.com/support)
- [GitHub Organization](https://github.com/densify-dev)
- [Terms of Service](https://www.densify.com/legal)
- [Privacy Policy](https://www.densify.com/privacy-policy)
- [Contact](https://www.densify.com/contact)
- [JSON-LD](json-ld/densify-context.jsonld) — [JSON-LD](https://www.w3.org/TR/json-ld11/)
- [Vocabulary](vocabulary/densify-vocabulary.yml)
- [Capabilities](capabilities/densify-capabilities.yml)
- [L L Ms Txt](https://api.densify.com/llms.txt)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
