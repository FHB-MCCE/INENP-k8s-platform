# INENP Kubernetes Platform

Dieses Repository ist der zentrale Einstiegspunkt für das Projekt
**INENP Kubernetes Platform**.

Ziel des Projekts ist eine Kubernetes-basierte Plattform auf Google Cloud
Platform. Die Infrastruktur wird mit Terraform bereitgestellt, Plattform- und
Anwendungsressourcen werden über GitOps synchronisiert, und tenant-spezifische
App-Instanzen werden deklarativ über Crossplane erzeugt.

## Projektziel

Die Plattform stellt eine reproduzierbare Umgebung für eine tenant-basierte
3-Tier-Anwendung auf Kubernetes bereit.

Jeder Tenant erhält eine isolierte Anwendungsinstanz mit:

- Frontend,
- Backend API,
- Datenbank,
- eigenem Kubernetes-Namespace,
- tenant-spezifischer Konfiguration und Secrets,
- DNS- und HTTPS-Endpunkten,
- Ressourcen- und Netzwerkisolation.

Die Demo zeigt, wie ein neuer Tenant durch einen GitOps-Claim angelegt wird und
wie Argo CD, Crossplane und die Plattform-Operatoren danach automatisch die
benötigten Ressourcen bereitstellen.

## Repositories

| Repository | Sichtbarkeit | Zweck |
|---|---|---|
| `INENP-k8s-platform` | öffentlich | zentrale Dokumentation, Projektübersicht, Projektplan, Kosten, Runbooks und Präsentation |
| `INENP-k8s-platform-iac` | öffentlich | Terraform für GCP, GKE, IAM, DNS, Secret Manager und Plattformzugriff |
| `INENP-k8s-platform-gitops` | öffentlich | GitOps-Konfiguration, Argo-CD-Applications, Plattform-Komponenten, Crossplane-Ressourcen und Tenant Claims |
| `INENP-k8s-platform-backend` | öffentlich | Backend-Anwendung, Docker Image, GHCR Publishing und Helm Chart |
| `INENP-k8s-platform-frontend` | privat | Frontend-Anwendung, privates Docker Image, GHCR Publishing und Helm Chart |

## Architekturüberblick

Die Plattform ist in zwei Phasen getrennt: Day 1 Plattform-Bootstrap und Day 2
Tenant-Provisioning.

Day 1 erstellt die Basisplattform:

- Google-Cloud-Infrastruktur,
- GKE Cluster,
- IAM und Workload Identity,
- Cloud DNS Zone,
- Google Secret Manager,
- Argo CD,
- Plattform-Operatoren wie cert-manager, ExternalDNS, External Secrets
  Operator, Crossplane und CloudNativePG.

Day 2 verwendet diese Plattform für tenant-spezifische Anwendungsinstanzen:

- Tenant Claims werden in das GitOps-Repository committed,
- Argo CD synchronisiert den gewünschten Zustand in den Cluster,
- Crossplane erzeugt die benötigten Kubernetes-Ressourcen,
- CloudNativePG stellt tenant-spezifische Datenbanken bereit,
- Helm Charts deployen Backend und Frontend,
- External Secrets Operator stellt Secrets aus Google Secret Manager bereit,
- ExternalDNS erzeugt DNS-Einträge,
- cert-manager stellt öffentliche HTTPS-Zertifikate aus.

## Tech-Stack

| Bereich | Technologie |
|---|---|
| Cloud Provider | Google Cloud Platform |
| Kubernetes | GKE Standard |
| Infrastructure as Code | Terraform |
| GitOps | Argo CD |
| Tenant-Provisioning | Crossplane |
| Secret Management | External Secrets Operator + Google Secret Manager |
| DNS-Management | ExternalDNS + Cloud DNS |
| TLS-Zertifikate | cert-manager + Let's Encrypt DNS-01 |
| Datenbank | CloudNativePG |
| Container Registry | GitHub Container Registry |
| Anwendungsdeployment | Helm Charts |
| CI/CD | GitHub Actions |

## Projektplanung und Gates

Der ursprüngliche Projektplan wurde mit dem tatsächlichen Umsetzungsstand
abgeglichen und als Markdown-Dokument in diesem Repository abgelegt:

- [Projektplan und Gate-Abgleich](docs/project-plan-gates.md)

Der Plan dokumentiert Gate 0 bis Gate 7, die wichtigsten Nachweise, bewusste
Abweichungen und die abschließende Einordnung der Teamverteilung.

## Day 1: Plattform-Bootstrap

Day 1 erzeugt die Plattformgrundlage.

Umgesetzte Bestandteile:

- GCP- und GKE-Ressourcen mit Terraform,
- IAM, Service Accounts, OIDC und Workload Identity,
- DNS-Zone und Secret-Manager-Container,
- Argo-CD-Bootstrap,
- Plattform-Operatoren über GitOps,
- Lecturer-Zugangsprozess,
- dokumentierte Ausnahmen wie DNS-Delegation und manuell bereitzustellende
  Secret-Werte.

Die DNS- und Secret-Manager-Grundlage ist dokumentiert unter
[DNS und Secret Manager Foundation](docs/dns-and-secrets.md).

## Day 2: Tenant-Provisioning

Day 2 erzeugt und betreibt tenant-spezifische Anwendungsinstanzen.

Umgesetzte Bestandteile:

- `TenantApplication` Crossplane API,
- Tenant Claims über GitOps,
- ein Namespace pro Tenant,
- tenant-spezifische Datenbankressourcen,
- Backend und Frontend über Helm,
- DNS- und HTTPS-Endpunkte,
- NetworkPolicies, ResourceQuotas und LimitRanges,
- Update-Pfad zuerst über Staging, danach Demo/Prod.

Ein neuer Tenant wird durch einen `TenantApplication` Claim im GitOps-Repository
angelegt. Nach dem Merge synchronisiert Argo CD automatisch, und Crossplane
erstellt die benötigten Ressourcen.

## Kapazitäts- und Kostenplanung

Kapazitäts- und Kostenplanung werden im zentralen GitHub Issue gepflegt:

- [Capacity & Cost Planning #1](https://github.com/FHB-MCCE/INENP-k8s-platform/issues/1)

Die Plattform nutzt ein kostenbewusstes GKE-Standard-Setup. Die ursprüngliche
Monatsplanung wurde mit den tatsächlichen GCP-Kosten verglichen. Die realen
Kosten liegen unter der Planung, hauptsächlich wegen kürzerer aktiver Laufzeit,
CloudNativePG im Cluster statt Cloud SQL und geringer Netzwerk-/DNS-/Secret-
Manager-Nutzung.

Weitere Details stehen in [Kostenstand und Abweichungsanalyse](docs/cost-tracking.md).

## Beitragsworkflow

Alle Arbeiten sind über GitHub nachvollziehbar.

Regeln:

- für relevante Änderungen ein GitHub Issue anlegen,
- pro Issue oder eng geschnittenem Teil ein Feature-Branch,
- jede Änderung über Pull Request nach `main`,
- Conventional Commits verwenden,
- Commit und PR mit der passenden Issue-Nummer referenzieren,
- keine Merge Commits,
- Reviews vor dem Merge,
- Dokumentation bei Implementierungsänderungen aktuell halten.

Beispiel:

```text
docs(platform): add final gate project plan (#26)
```

## Aktueller Stand

Die Plattform ist für die Abgabe vorbereitet. GKE, Argo CD, Crossplane,
ExternalDNS, cert-manager, External Secrets Operator, CloudNativePG, Backend und
Frontend sind in den Runbooks und Nachweisen dokumentiert.

Referenzendpunkte:

| Tenant | Frontend | Backend Health |
|---|---|---|
| Demo | `https://demo.inenp.naehrer.me/` | `https://api.demo.inenp.naehrer.me/actuator/health` |
| Staging | `https://staging.inenp.naehrer.me/` | `https://api.staging.inenp.naehrer.me/actuator/health` |
| Prod | `https://prod.inenp.naehrer.me/` | `https://api.prod.inenp.naehrer.me/actuator/health` |

Der zusätzliche Prod-Tenant wurde über einen GitOps-PR angelegt und dient als
weiterer Nachweis für deklaratives Tenant-Provisioning.

## Wichtige Dokumentation

- [Projektplan und Gate-Abgleich](docs/project-plan-gates.md)
- [E2E Tenant Provisioning Runbook](docs/e2e-tenant-runbook.md)
- [Kostenstand und Abweichungsanalyse](docs/cost-tracking.md)
- [Tenant Contribution Guide / Anleitung für neue Tenants](docs/tenant-contribution-guide.md)
- [Präsentationsübersicht](docs/presentation-outline.md)
- [Qualitäts- und Sicherheitsaudit](docs/quality-security-audit.md)
- [DNS und Secret Manager Foundation](docs/dns-and-secrets.md)
- [KI-Nutzung](docs/ai-usage.md)

## Deadline

Die Abgabefrist ist:

```text
Freitag, 26. Juni 2026, 14:00 CEST
```

Nach der Deadline sollen keine Änderungen mehr vorgenommen werden.
