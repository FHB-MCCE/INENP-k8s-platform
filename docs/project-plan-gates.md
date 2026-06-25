# Projektplan und Gate-Abgleich

Dieses Dokument fasst den finalen Projektplan für die INENP Kubernetes Platform
zusammen und gleicht ihn mit der tatsächlich umgesetzten Lösung ab.

Der ursprüngliche Plan wurde lokal als `Projektplan.pdf` geführt. Die finale
Abgabe nutzt dieses Dokument als GitHub-nachvollziehbare Markdown-Fassung.

## Zielbild

Bis zur Abgabe am 26. Juni 2026 sollte eine GKE-basierte Plattform entstehen,
bei der ein neuer Tenant deklarativ über GitOps und Crossplane angelegt wird.
Der Tenant erhält automatisch:

- einen eigenen Kubernetes-Namespace,
- ResourceQuota, LimitRange und NetworkPolicy,
- eine tenant-spezifische CloudNativePG-Datenbank,
- Backend und Frontend über Helm,
- Secret-Wiring über External Secrets Operator und Google Secret Manager,
- DNS-Einträge über ExternalDNS,
- HTTPS-Zertifikate über cert-manager,
- reproduzierbare Updates zuerst über Staging und danach über Demo/Prod.

Dieses Zielbild wurde erreicht. Der tatsächliche Verlauf wich an einigen
Stellen vom ursprünglichen Plan ab; diese Abweichungen sind unten dokumentiert.

## Verwendeter Stack

| Bereich | Umsetzung |
|---|---|
| Cloud | Google Cloud Platform |
| Kubernetes | GKE Standard, zonal in `europe-west3-a` |
| IaC | Terraform |
| GitOps | Argo CD |
| Tenant-Provisioning | Crossplane `TenantApplication` XRD und Composition |
| DNS | Cloud DNS + ExternalDNS |
| TLS | cert-manager + Let's Encrypt |
| Secrets | Google Secret Manager + External Secrets Operator |
| Datenbank | CloudNativePG im Cluster |
| Container Registry | GitHub Container Registry |
| Backend | öffentliches GHCR Image + Helm Chart |
| Frontend | privates GHCR Image + Helm Chart |
| CI/CD | GitHub Actions, Quality Gates und PR-Reviews |

## Gate-Übersicht

| Gate | Ursprünglicher Plan | Tatsächlicher Stand | Abweichung / Entscheidung |
|---|---|---|---|
| Gate 0 | Repos, Issues, Cost Planning, Domain-Plan, Verified-PR-Klärung und App-Forks vorbereiten | Repos wurden angelegt, Issues und PR-Workflow etabliert, Cost Planning in Issue #1 dokumentiert, Apps vorbereitet | Verified Commits wurden nicht als zusätzliche technische Pflicht behandelt; GitHub-Verified/Squash-PRs und geprüfte Commit-Mails wurden als ausreichend bewertet |
| Gate 1 | Terraform-Grundstruktur, VPC, Subnet, GKE, IAM, DNS, Secret Manager und IaC-CI | Terraform-Projekt, Netzwerk, GKE, DNS-Zone, Secret-Container, IAM/Workload Identity und IaC-Checks umgesetzt | Der vollständige One-Click-Apply-Workflow kam später in Gate 6 dazu, nicht bereits zu Beginn |
| Gate 2 | GitOps Foundation mit Argo CD, cert-manager, ExternalDNS, ESO, CloudNativePG und Crossplane | Argo CD App-of-Apps, Plattform-Operatoren und Crossplane wurden über GitOps verwaltet | Crossplane GCP Provider und Dummy-Claim wurden nicht beibehalten; stattdessen wurden die tatsächlich benötigten Kubernetes-/Helm-Provider und echte Tenant-Claims umgesetzt |
| Gate 3 | Backend/Frontend Dockerfiles, CI, GHCR, Helm Charts und Schnittstellen | Backend und Frontend bauen Images, veröffentlichen in GHCR und besitzen Helm Charts | Frontend bleibt privat; Pull-Zugriff erfolgt über Secret-Wiring |
| Gate 4 | `TenantApplication` XRD, Claim-Schema, Composition, Staging/Demo Claims, Quotas, NetworkPolicies, DB, Backend, Frontend, DNS und TLS | `TenantApplication` API und Composition erzeugen die benötigten Tenant-Ressourcen; Staging, Demo und später Prod existieren als Claims | Der Umfang wurde über mehrere PRs stabilisiert; echte End-to-End-Verifikation erfolgte in den folgenden Gates |
| Gate 5 | E2E-Stabilisierung, Tenant-Runbook, Kostenvergleich, Update-/Rollback-Prozess | E2E-Runbook, Tenant-Checks, Backend-/Frontend-Verifikation, Kostenvergleich und Staging-zu-Demo-Freigabe dokumentiert | API-Vertrag musste über Staging korrigiert und danach nach Demo freigegeben werden |
| Gate 6 | Dokumentation, One-Click-Apply, Demo-Freeze, README-Finalisierung | IaC `Apply Infrastructure` Workflow, OIDC, Runbooks, Quality/Security Audit und Demo-Freeze dokumentiert | Der Gate-6-Scope wurde erweitert, weil der One-Click-/No-Manual-Click-Nachweis explizit aus der Aufgabenstellung nachgezogen wurde |
| Gate 7 | Abgabeprüfung, finale Kosten, Actions, Argo CD Health, Demo und Präsentation | Präsentation, Demo-Runbook, Kostenstand, Prod-Tenant und Issue-/PR-Hygiene wurden finalisiert | Monitoring wurde bewusst nicht als Bonusziel umgesetzt; als Präsentationsthema wurde Multi-Tenancy & Security gewählt |

## Gate 0: Projektstart

Umgesetzt:

- zentrale Projektübersicht im Platform-Repository,
- Capacity-&-Cost-Planning Issue,
- Repository-Übersicht und Contribution-Regeln,
- getrennte Repositories für IaC, GitOps, Backend und Frontend,
- privates Frontend-Repository gemäß Aufgabenstellung,
- Dokumentation der KI-Nutzung.

Nachweise:

- [Capacity & Cost Planning #1](https://github.com/FHB-MCCE/INENP-k8s-platform/issues/1)
- [Project Overview #2](https://github.com/FHB-MCCE/INENP-k8s-platform/issues/2)
- [AI Usage #10](https://github.com/FHB-MCCE/INENP-k8s-platform/issues/10)

## Gate 1: Infrastructure Bootstrap

Umgesetzt:

- Terraform-Struktur mit GCS-State-Konzept,
- VPC und Subnet,
- GKE Standard Cluster `inenp-dev-gke`,
- Node Pool `inenp-dev-gke-primary`,
- Autoscaling von 3 bis 4 `n2-standard-2` Nodes,
- Cloud DNS Zone,
- Secret Manager Container,
- IAM, Workload Identity und GitHub OIDC,
- Lecturer-Zugangsprozess.

Nachweise:

- [IaC Netzwerk #3](https://github.com/FHB-MCCE/INENP-k8s-platform-iac/issues/3)
- [GKE Cluster #4](https://github.com/FHB-MCCE/INENP-k8s-platform-iac/issues/4)
- [Lecturer Access #5](https://github.com/FHB-MCCE/INENP-k8s-platform-iac/issues/5)
- [One-Click Apply #17](https://github.com/FHB-MCCE/INENP-k8s-platform-iac/issues/17)

## Gate 2: GitOps Foundation

Umgesetzt:

- GitOps-Repository-Struktur,
- Argo CD Bootstrap und Root Application,
- cert-manager und ClusterIssuer,
- ExternalDNS,
- External Secrets Operator,
- CloudNativePG Operator,
- Crossplane Core und Provider.

Abweichung:

Der ursprüngliche Plan enthielt einen frühen Dummy-Claim und einen GCP Provider.
Beides wurde nicht als finaler Weg beibehalten. Die Plattform nutzt stattdessen
die tatsächlich benötigten Kubernetes- und Helm-Provider. Der Nachweis erfolgt
über echte `TenantApplication` Claims.

Nachweise:

- [Argo CD Bootstrap](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/blob/main/docs/argocd-bootstrap.md)
- [Crossplane Provider #9](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/issues/9)
- [ExternalDNS](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/blob/main/docs/external-dns.md)
- [External Secrets](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/blob/main/docs/external-secrets.md)

## Gate 3: App Build und Helm Charts

Umgesetzt:

- Backend Dockerfile, CI, GHCR Publish und Helm Chart,
- Frontend Dockerfile, CI, privates GHCR Image und Helm Chart,
- Runtime-Konfiguration für Backend-URL,
- Backend-Health-Checks,
- Pull-Secret-Konzept für das private Frontend-Image.

Nachweise:

- [Backend Repository](https://github.com/FHB-MCCE/INENP-k8s-platform-backend)
- [Frontend Repository](https://github.com/FHB-MCCE/INENP-k8s-platform-frontend)
- [Frontend Image Pull Secret](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/blob/main/docs/frontend-image-pull-secret.md)

## Gate 4: Crossplane Tenant-Provisioning

Umgesetzt:

- `TenantApplication` XRD,
- Claim-Schema für Tenant-Name, Environment, Hostname, Image-Tags und Datenbankgröße,
- Composition für Namespace, Quota, Limits, NetworkPolicy, Secrets, Datenbank, Backend und Frontend,
- Staging- und Demo-Tenant-Claims,
- später zusätzlicher Prod-Tenant-Claim.

Nachweise:

- [TenantApplication API #10](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/issues/10)
- [Tenant Claims #11](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/issues/11)
- [Prod Tenant PR #51](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/pull/51)
- [`tenants/prod/tenant-application.yaml`](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/blob/main/tenants/prod/tenant-application.yaml)

## Gate 5: End-to-End-Stabilisierung

Umgesetzt:

- E2E Tenant Provisioning Runbook,
- Cluster-Checks für Argo CD, TenantApplications, Pods, Ingress und Zertifikate,
- HTTPS-Smoke-Tests,
- Backend-/Datenbank-Verifikation,
- Frontend-/TLS-Verifikation,
- Staging-Canary für API-Vertrag,
- Promotion der geprüften Image-SHAs nach Demo.

Abweichung:

Der API-Vertrag zwischen Frontend und Backend musste während der Stabilisierung
korrigiert werden. Die Änderung wurde bewusst zuerst in Staging geprüft und erst
danach nach Demo übernommen.

Nachweise:

- [E2E Tenant Runbook #11](https://github.com/FHB-MCCE/INENP-k8s-platform/issues/11)
- [API Contract Rollout #47](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/issues/47)
- [Backend Verification](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/blob/main/docs/backend-database-verification.md)
- [Frontend TLS Verification](https://github.com/FHB-MCCE/INENP-k8s-platform-gitops/blob/main/docs/frontend-tls-update-verification.md)

## Gate 6: Dokumentation und Demo-Freeze

Umgesetzt:

- zentrale README finalisiert,
- README-Dateien und Runbooks in Backend und Frontend finalisiert,
- One-Click-Apply-Workflow dokumentiert,
- Quality Gates in allen Repos vorbereitet,
- Kosten- und Abweichungsanalyse dokumentiert,
- Multi-Tenancy & Security als Präsentationsthema gewählt,
- Präsentation erstellt.

Abweichung:

Der One-Click-/No-Manual-Click-Nachweis wurde im Verlauf explizit stärker
gemacht als im ursprünglichen Projektplan. Der endgültige Nachweis trennt:

1. Plattform-Bootstrap über GitHub Actions `Apply Infrastructure`.
2. Tenant-Provisioning über Merge eines `TenantApplication` Claims.

Nachweise:

- [E2E Tenant Runbook](e2e-tenant-runbook.md)
- [Cost Tracking](cost-tracking.md)
- [Quality and Security Audit](quality-security-audit.md)
- [Tenant Contribution Guide](tenant-contribution-guide.md)

## Gate 7: Abgabe und Präsentation

Umgesetzt:

- finale Issue-/PR-Hygiene,
- aktualisierter Kostenstand im Capacity-Planning-Issue,
- Präsentationsdeck mit Sprechertexten,
- Demo-Runbook,
- finale Tenant-Demo mit GitOps/Crossplane,
- zusätzlicher Prod-Tenant als GitOps-Nachweis.

Nicht umgesetzt:

- Monitoring wurde nicht als Bonusziel umgesetzt.

Begründung:

Monitoring war im ursprünglichen Projektplan bewusst nicht als realistisches
Bonusziel eingeplant. Die Präsentation wählt stattdessen das laut Aufgabenstellung
zulässige Zusatzthema Multi-Tenancy & Security.

## Planabweichungen im Überblick

| Abweichung | Bewertung |
|---|---|
| Crossplane GCP Provider nicht final genutzt | Keine fachliche Lücke, da Tenant-Ressourcen über Kubernetes, Helm, ESO, CNPG und GitOps erzeugt werden |
| Dummy-Claim nicht final umgesetzt | Durch echte `TenantApplication` Claims ersetzt |
| One-Click-Apply später als geplant | Gate 6 hat den IaC-Apply-Workflow und OIDC-Nachweis nachgezogen |
| Prod-Tenant zusätzlich erstellt | Positiver zusätzlicher Nachweis für Tenant-Provisioning per GitOps |
| Monitoring nicht umgesetzt | Bewusste Entscheidung, da Bonus; Präsentation nutzt Multi-Tenancy & Security |
| Kosten niedriger als Planung | Erwartbar durch kürzere Laufzeit, CloudNativePG statt Cloud SQL und geringe Netzwerk-/Secret-Nutzung |

## Team-Verteilung

Die Arbeit wurde entlang der vier Pillars verteilt:

| Person | Schwerpunkt |
|---|---|
| Jan | zentrale Dokumentation, Cost/Capacity, Terraform-Kern, GKE, Crossplane API, Tenant Claims, E2E-Runbook |
| Sebastian | Repo-Hygiene, CI/Quality Gates, IAM, OIDC, Workload Identity, ExternalDNS, Security-Review |
| Martin | Backend, CloudNativePG, Datenbank, Secret Manager, Backend-Betrieb, Kostenbelege |
| Maximilian | Argo CD, cert-manager/TLS, Frontend, private GHCR Pulls, Runtime Config, Update-/Rollback-Pfad |

Die Präsentationsnote ist laut Aufgabenstellung eine Ausnahme von der
Gleichverteilung und gilt gemeinsam für alle Teammitglieder.

## KI-Nutzung

KI wurde unterstützend für Strukturierung, Projektplan-Abgleich,
Code-/Konfigurationsentwürfe, Dokumentationsformulierung, Issue-Hygiene und
Review-Hinweise genutzt. Alle KI-Ergebnisse wurden vom Team geprüft, mit den
tatsächlichen GitHub-Issues, Pull Requests und Repository-Inhalten abgeglichen
und nur nach menschlicher Kontrolle übernommen.
