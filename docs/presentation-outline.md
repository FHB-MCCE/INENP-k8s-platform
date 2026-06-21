# Präsentationsoutline

Maximale Dauer: 20 Minuten.

## Zeitplan

| Abschnitt | Person | Zeit | Inhalt |
|---|---|---:|---|
| Einstieg und Zielbild | Jan | 2 min | Projektziel, Repositories, Day 1 / Day 2 |
| Architektur und Kosten | Jan | 3 min | GCP, GKE, GitOps, Crossplane, Kostenplanung |
| Bootstrap und Security | Sebastian | 4 min | Terraform, OIDC, Argo CD, Secret-Grenzen, Quality Gates |
| Backend und Datenbank | Martin | 4 min | Spring Boot, CloudNativePG, Flyway, METAR-Proxy, Health Checks |
| Frontend, TLS und Update | Maximilian | 4 min | Quasar, private GHCR Pulls, Runtime Config, DNS/TLS, Staging Rollout |
| Demo und Ausblick | alle | 3 min | Tenant Claim, HTTPS App, Skalierung auf weitere Tenants |

## Demo-Reihenfolge

1. GitHub Actions One-Click Apply zeigen.
2. GitOps Tenant Claims zeigen.
3. Argo CD Applications zeigen.
4. Demo und Staging HTTPS öffnen.
5. User, Favorite Location, Forecast und METAR testen.
6. Pods, Zertifikate und TenantApplications zeigen.
7. Staging Update/Rollback erklären.

## Pflichtaussagen

- Infrastruktur entsteht über Terraform.
- Plattform- und App-Ressourcen werden über GitOps synchronisiert.
- Crossplane stellt Tenant-Ressourcen bereit.
- Secrets werden über Google Secret Manager und External Secrets Operator
  eingebunden.
- Frontend Image ist privat, Backend Image öffentlich nutzbar.
- AVWX-Key bleibt serverseitig.
- Kosten wurden geplant und mit Ist-Kosten verglichen.

## Ausblick auf Skalierung

- Neue Tenants entstehen über weitere Claims.
- Staging bleibt der sichere Update-Pfad.
- Mehr Tenants brauchen klare Ressourcenlimits und eventuell größere Node Pools.
- Monitoring wäre der sinnvollste Bonus-Ausbau nach Abgabe.

