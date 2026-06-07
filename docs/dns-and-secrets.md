# DNS And Secret Manager Foundation

This document captures the Gate 1 decisions for DNS delegation and Google Secret Manager. It describes the intended platform flow before the first Terraform apply in Gate 2.

## Domain

The platform domain is:

```text
inenp.naehrer.me
```

The parent domain `naehrer.me` is owned by the team. No registrar-side delegation records are configured yet. Terraform will create a public Cloud DNS managed zone for `inenp.naehrer.me`; after the zone exists, the generated name server records must be copied manually to the DNS settings of `naehrer.me`.

## Cloud DNS Flow

Day 1 and Gate 2 use this sequence:

1. Terraform creates the Cloud DNS managed zone for `inenp.naehrer.me`.
2. Terraform outputs the authoritative Cloud DNS name servers.
3. The domain owner adds those name servers as delegation records for `inenp.naehrer.me` in the parent zone of `naehrer.me`.
4. ExternalDNS later receives Workload Identity permissions for the managed zone.
5. Tenant ingress records are created automatically by ExternalDNS after the GitOps components are installed.

The delegation step is intentionally manual because it happens outside the Google Cloud project.

## Expected DNS Records

The platform expects DNS records below the delegated zone, for example:

| Record | Owner | Purpose |
|---|---|---|
| `*.inenp.naehrer.me` | ExternalDNS | Tenant and application ingress records |
| `argocd.inenp.naehrer.me` | ExternalDNS or manual bootstrap | Argo CD UI after bootstrap |
| `api.<tenant>.inenp.naehrer.me` | ExternalDNS | Tenant backend endpoint |
| `<tenant>.inenp.naehrer.me` | ExternalDNS | Tenant frontend endpoint |

Exact tenant names are defined later through GitOps tenant claims.

## Secret Manager Foundation

Google Secret Manager is the source of truth for platform and tenant secrets. Kubernetes workloads do not store secret values in Git. External Secrets Operator reads values from Secret Manager and materializes Kubernetes Secrets in the target namespaces.

Gate 1 Terraform only creates empty Secret Manager secret containers. It does not create secret versions and does not commit secret values.

Initial secret containers:

| Secret ID | Intended Consumer | Purpose |
|---|---|---|
| `frontend-ghcr-pull-token` | GitOps image pull secret wiring | Pull private frontend images from GHCR |
| `database-app-password` | CloudNativePG / tenant backend | Initial application database password placeholder |
| `backend-jwt-secret` | Backend application | Token signing secret placeholder |
| `letsencrypt-account-email` | cert-manager automation | ACME account contact placeholder |

Secret values are inserted manually or through a controlled bootstrap step after the Secret Manager resources exist.

## Open Items

- Confirm registrar access for `naehrer.me`.
- Apply the Cloud DNS zone in Gate 2 and record the generated name servers.
- Add the parent-zone delegation records after Terraform creates the zone.
- Define the final ExternalDNS service account and IAM binding in the GitOps and IaC work items.
- Add real secret versions only after the team agrees on bootstrap ownership.
