# E2E Tenant Provisioning Runbook

Dieses Runbook beschreibt den reproduzierbaren Gate-7-Demoablauf für die INENP
Kubernetes Platform.

## Voraussetzungen

- GCP-Projekt: `dulcet-velocity-495612-j0`
- GKE Cluster: `inenp-dev-gke`
- Zone: `europe-west3-a`
- Domain: `inenp.naehrer.me`
- Argo CD Root Application ist installiert
- Secret-Werte sind in Google Secret Manager gepflegt
- Parent-DNS-Delegation für `inenp.naehrer.me` ist gesetzt

Keine Secret-Payloads in Terminals, Screenshots, Issues oder Dokumentation
ausgeben.

## Repositories

| Repository | Zweck |
|---|---|
| `INENP-k8s-platform` | Zentrale Doku, Runbook, Kosten und Präsentation |
| `INENP-k8s-platform-iac` | Terraform, GKE, IAM, DNS, Secret Manager, One-Click Apply |
| `INENP-k8s-platform-gitops` | Argo CD Apps, Crossplane API, Tenant Claims |
| `INENP-k8s-platform-backend` | Backend App, Docker Image, Helm Chart |
| `INENP-k8s-platform-frontend` | Frontend App, privates Docker Image, Helm Chart |

## One-Click Bootstrap

Der bevorzugte Bootstrap-Pfad ist der GitHub Actions Workflow
`Apply Infrastructure` im IaC-Repository.

Nachweis:

```text
IaC Actions Run: 27880688957
```

Ablauf:

1. Workflow auf `main` per `workflow_dispatch` starten.
2. Eingabe `APPLY` bestätigen.
3. GitHub authentifiziert sich per OIDC gegen GCP.
4. Terraform führt Plan und Apply aus.
5. Bootstrap-Script installiert oder aktualisiert Argo CD.
6. Root Application wird angewendet.
7. Argo CD synchronisiert Plattform- und Tenant-Ressourcen.

Dokumentierte externe Glue Points:

- initialer OIDC-Vertrauensanker
- Parent-DNS-Delegation
- kundenseitig bereitgestellte Secret-Werte in Google Secret Manager
- Lecturer-Zugangsprüfung mit eigener institutioneller Identität

## Tenant Claims

Tenant Claims liegen im GitOps-Repository:

```text
tenants/demo/tenant-application.yaml
tenants/staging/tenant-application.yaml
```

Demo und Staging sind die Gate-7-Referenzinstanzen. Beide verwenden geprüfte,
unveränderliche Image-SHAs:

| Komponente | Image-SHA |
|---|---|
| Backend | `f5feb13942519da31cfac8f87ebdd58fd0cc0784` |
| Frontend | `2c3e96c3f4962723d823e6b09da0ef521f5cc0d9` |

## Cluster Checks

```sh
kubectl -n argocd get applications
kubectl -n tenants get tenantapplications
kubectl get ingress -A
kubectl -n tenant-demo get pods
kubectl -n tenant-staging get pods
kubectl -n tenant-demo get certificate
kubectl -n tenant-staging get certificate
```

Erwartung:

- Root-App und konkrete Tenant-Applications sind gesund.
- `demo` und `staging` sind `SYNCED=True` und `READY=True`.
- Frontend, Backend und Datenbank laufen in beiden Tenant-Namespaces.
- TLS-Zertifikate sind `READY=True`.
- Ingresses haben die ingress-nginx LoadBalancer-IP `34.179.249.49`.

Bekannte Besonderheit: `tenant-application-api` kann wegen von Crossplane
ergänzten Feldern `OutOfSync / Healthy` anzeigen. Das ist kein Blocker, solange
Root-App, Claims und Tenant-Anwendungen gesund sind.

## HTTPS Smoke Tests

```sh
curl -sS -o /dev/null -w 'demo_frontend %{http_code}\n' https://demo.inenp.naehrer.me/
curl -sS -o /dev/null -w 'staging_frontend %{http_code}\n' https://staging.inenp.naehrer.me/
curl -sS -o /dev/null -w 'demo_api %{http_code}\n' https://api.demo.inenp.naehrer.me/actuator/health
curl -sS -o /dev/null -w 'staging_api %{http_code}\n' https://api.staging.inenp.naehrer.me/actuator/health
curl -sS -o /dev/null -w 'demo_users %{http_code}\n' https://api.demo.inenp.naehrer.me/api/user/
curl -sS -o /dev/null -w 'staging_users %{http_code}\n' https://api.staging.inenp.naehrer.me/api/user/
```

Erwartung: Alle Smoke Tests liefern HTTP `200`.

## Funktionaler Demoablauf

1. Demo Frontend öffnen: `https://demo.inenp.naehrer.me/`
2. Staging Frontend öffnen: `https://staging.inenp.naehrer.me/`
3. Benutzerliste laden.
4. Benutzer anlegen oder vorhandenen Benutzer auswählen.
5. Favorite Location anlegen.
6. Forecast für die Location laden.
7. METAR über den Backend-Endpunkt laden.
8. Im Browser prüfen, dass kein AVWX-Key ausgeliefert wird.

## Update und Rollback

Update-Pfad:

1. Neues Image wird gebaut und in GHCR veröffentlicht.
2. Staging Tenant Claim oder Composition-Wert wird auf den neuen SHA gesetzt.
3. Argo CD synchronisiert Staging.
4. Staging wird per HTTPS und Funktionstest geprüft.
5. Derselbe SHA wird nach Demo promoviert.

Rollback:

1. Vorherigen SHA aus Git-Historie oder PR übernehmen.
2. Staging zurücksetzen und prüfen.
3. Demo nur ändern, wenn Staging erfolgreich ist.

## Lecturer Access

Der Lecturer-Zugang ist im IaC-Repository unter
`docs/lecturer-cluster-access.md` dokumentiert. Die abschließende Prüfung
erfolgt mit der eigenen institutionellen Identität des Lehrenden:

```sh
kubectl auth can-i '*' '*' --all-namespaces
```

Erwartung: `yes`.

## Demo-Probe

Vor Abgabe zweimal durchführen:

| Probe | Datum | Ergebnis | Notizen |
|---|---|---|---|
| 1 | offen | offen | vor Präsentation ausfüllen |
| 2 | offen | offen | vor Präsentation ausfüllen |

