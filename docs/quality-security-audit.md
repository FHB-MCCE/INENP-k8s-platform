# Quality und Security Audit

Dieses Dokument fasst den Gate-7-Audit für Repository-Hygiene, CI, IAM und
Secret-Grenzen zusammen.

## Repository-Hygiene

| Check | Erwartung |
|---|---|
| Pull Requests | Jede relevante Änderung läuft über PR. |
| Merge-Strategie | Squash Merge, keine Merge Commits. |
| Commit-Format | Conventional Commits mit Issue-Referenz. |
| Branches | Feature-/Fix-Branches nach Merge löschen. |
| Reviews | Mindestens ein anderes Teammitglied reviewed. |
| CI | Repos haben passende Validierung vor Merge. |

## CI und Actions

| Repo | Relevante Checks |
|---|---|
| IaC | Terraform fmt/validate und geschützter Apply Workflow |
| GitOps | YAML/Manifest/Helm-Validierung |
| Backend | Gradle Tests, Boot Jar, Docker Build, GHCR Publish |
| Frontend | Lint, Unit Tests, Quasar Build, Docker Build, privater GHCR Publish |
| Platform | Markdown-/Doku-Änderungen über PR und Review |

## IAM und OIDC

Der IaC Apply Workflow nutzt GitHub OIDC statt statischer GCP-Schlüssel. Der
Trust ist auf Repository, Branch und manuell gestartete Runs begrenzt. Der
initiale OIDC-Vertrauensanker ist als dokumentierte Bootstrap-Ausnahme
festgehalten.

## Secret-Grenzen

| Secret-Bereich | Regel |
|---|---|
| Google Secret Manager | Quelle für Secret-Werte |
| External Secrets Operator | Synchronisiert Werte in Kubernetes |
| GitOps | Enthält nur Secret-Namen und Referenzen |
| GHCR Frontend Pull | Separates Pull Secret pro Tenant-Namespace |
| Argo CD Frontend Repo | Separates Repository Credential für private Helm-Quelle |
| AVWX | Nur Backend liest den Key serverseitig |

Nicht erlaubt:

- Secret-Payloads in Git
- Secret-Werte in Issues oder PRs
- dekodierte Kubernetes Secrets in Screenshots
- AVWX-Key im Browser oder Frontend Bundle

## Bekannte Ausnahmen

| Ausnahme | Begründung |
|---|---|
| Parent-DNS-Delegation | Externer Domain-Provider, nicht durch Terraform im GCP-Projekt steuerbar |
| initialer OIDC-Trust | Einmaliger Vertrauensanker, danach Terraform-verwaltet |
| kundenseitige Secret-Werte | Tokens/API-Keys müssen extern bereitgestellt werden |
| Lecturer Access Prüfung | Erfolgt mit persönlicher institutioneller Identität |

## Gate-7 Audit-Abschluss

Der Audit ist abgeschlossen, wenn:

- keine offenen Gate-7-PRs verbleiben
- CI der betroffenen Repos grün ist oder Doku-Änderungen nachvollziehbar sind
- offene Issues geschlossen oder begründet offen bleiben
- keine Secrets in Doku, Issues oder Logs enthalten sind
- Demo-Freeze dokumentiert ist

