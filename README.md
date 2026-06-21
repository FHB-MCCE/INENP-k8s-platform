# INENP Kubernetes Platform

This repository is the central entry point for the INENP Kubernetes platform project.

The goal of this project is to build and document a Kubernetes-based platform on Google Cloud Platform. The platform provisions its infrastructure with Infrastructure as Code, deploys platform and application resources through GitOps, and allows tenant-specific application instances to be created declaratively.

## Project Goal

The platform will provide a reproducible setup for running a tenant-based 3-tier application on Kubernetes.

Each tenant should receive an isolated application instance consisting of:

- a frontend
- a backend API
- a database
- a dedicated Kubernetes namespace
- tenant-specific configuration and secrets
- DNS and HTTPS endpoints
- resource and network isolation

The final demo should show how a new tenant can be created through GitOps and how the platform automatically provisions the required resources.

## Repository Overview

| Repository | Visibility | Purpose |
|---|---|---|
| `INENP-k8s-platform` | Public | Central documentation, project overview, planning, cost tracking and demo runbook |
| `INENP-k8s-platform-iac` | Public | Terraform code for GCP, GKE, IAM, DNS, Secret Manager and platform access |
| `INENP-k8s-platform-gitops` | Public | GitOps configuration, Argo CD applications, platform components, Crossplane resources and tenant claims |
| `INENP-k8s-platform-backend` | Public | Backend application, Docker image build, GHCR publishing and Helm chart |
| `INENP-k8s-platform-frontend` | Private | Frontend application, private Docker image build, GHCR publishing and Helm chart |

## Architecture Overview

The platform is split into two major phases: Day 1 platform bootstrap and Day 2 tenant provisioning.

Day 1 creates and configures the base platform:

- Google Cloud infrastructure
- GKE cluster
- IAM and Workload Identity
- Cloud DNS zone
- Google Secret Manager
- Argo CD
- platform operators such as cert-manager, ExternalDNS, External Secrets Operator, Crossplane and CloudNativePG

Day 2 uses the platform to create and operate tenant-specific application instances:

- tenant claims are committed to the GitOps repository
- Argo CD synchronizes the desired state to the cluster
- Crossplane creates the required Kubernetes resources
- CloudNativePG provides tenant-specific database resources
- Helm charts deploy backend and frontend
- External Secrets Operator injects required secrets
- ExternalDNS creates DNS records
- cert-manager issues HTTPS certificates

## Technology Stack

| Area | Technology |
|---|---|
| Cloud Provider | Google Cloud Platform |
| Kubernetes | GKE Standard |
| Infrastructure as Code | Terraform |
| GitOps | Argo CD |
| Tenant Provisioning | Crossplane |
| Secrets Management | External Secrets Operator + Google Secret Manager |
| DNS Management | ExternalDNS + Cloud DNS |
| TLS Certificates | cert-manager + Let's Encrypt DNS-01 |
| Database | CloudNativePG |
| Container Registry | GitHub Container Registry |
| Application Deployment | Helm charts |
| CI/CD | GitHub Actions |

## Day 1: Platform Bootstrap

Day 1 is responsible for creating the platform foundation.

Planned Day 1 tasks include:

- provisioning GCP and GKE resources with Terraform
- configuring IAM, service accounts and Workload Identity
- creating the DNS zone and Secret Manager resources
- bootstrapping Argo CD
- deploying platform operators through GitOps
- configuring access for the lecturer
- documenting all required manual steps and exceptions

The current DNS and Secret Manager foundation is documented in [DNS And Secret Manager Foundation](docs/dns-and-secrets.md).

AI-assisted planning and implementation support is documented in [AI Usage](docs/ai-usage.md).

After Day 1, the platform should exist and be ready to manage application resources declaratively.

## Day 2: Tenant Provisioning

Day 2 is responsible for using the platform to create tenant-specific application instances.

Planned Day 2 tasks include:

- defining a `TenantApplication` Crossplane API
- creating tenant claims through GitOps
- provisioning one namespace per tenant
- provisioning tenant-specific database resources
- deploying backend and frontend through Helm
- configuring DNS and HTTPS endpoints
- applying NetworkPolicies, ResourceQuotas and LimitRanges
- supporting application updates through a staging tenant first

After Day 2, a new tenant should be created by committing a tenant claim to the GitOps repository.

## Capacity and Cost Planning

Capacity and cost planning is tracked in the project issue:

- Capacity and Cost Planning: #1

The plan currently targets a cost-conscious GKE Standard setup on Google Cloud Platform. Actual costs will be tracked during the project and compared against the initial estimate before submission.

## Contribution Workflow

All project work should be traceable through GitHub.

Rules:

- create a GitHub issue for every relevant change
- create a feature branch for each issue
- open a pull request for every change
- use Conventional Commits
- reference the related issue in commits and pull requests
- avoid merge commits
- use reviews before merging
- keep documentation updated with implementation changes

Example commit message:

```text
docs(platform): add initial project overview (#2)
```

## Current Status

Gate 6 is technically complete and Gate 7 is the documentation and demo freeze. Demo and staging run in GKE, Argo CD reports the tenant frontend and backend applications as healthy, TLS certificates are ready and the public HTTPS endpoints respond.

Reference endpoints:

| Tenant | Frontend | Backend Health |
|---|---|---|
| Demo | `https://demo.inenp.naehrer.me/` | `https://api.demo.inenp.naehrer.me/actuator/health` |
| Staging | `https://staging.inenp.naehrer.me/` | `https://api.staging.inenp.naehrer.me/actuator/health` |

Gate-7 documentation:

- [E2E Tenant Provisioning Runbook](docs/e2e-tenant-runbook.md)
- [Cost Tracking and Deviation Analysis](docs/cost-tracking.md)
- [Tenant Contribution Guide](docs/tenant-contribution-guide.md)
- [Presentation Outline](docs/presentation-outline.md)
- [Quality and Security Audit](docs/quality-security-audit.md)
- [DNS and Secret Manager Foundation](docs/dns-and-secrets.md)
- [AI Usage](docs/ai-usage.md)
## Deadline

The submission deadline is:

```text
Friday, 26 June 2026, 14:00 CEST
```

No changes should be made after the deadline.
