# Tenant Contribution Guide

Dieses Dokument beschreibt, wie ein neuer Tenant vorbereitet, getestet und in
Demo übernommen wird.

## Grundprinzip

Ein Tenant wird nicht per Klick im Cluster erzeugt. Der gewünschte Zustand wird
im GitOps-Repository beschrieben. Argo CD synchronisiert die Änderung,
Crossplane erzeugt die benötigten Ressourcen.

## Neue Tenant-Instanz

1. Im GitOps-Repository einen neuen Tenant Claim anlegen.
2. Tenant-Namen, Hostname, Environment und Image-SHAs setzen.
3. PR öffnen und validieren lassen.
4. Review einholen.
5. Nach Merge synchronisiert Argo CD den Claim.
6. Crossplane erstellt Namespace, Backend, Frontend, Datenbank, Secrets,
   Ingress und TLS-Ressourcen.

## Werte, die ein Tenant braucht

| Feld | Zweck |
|---|---|
| `tenantName` | stabiler Tenant-Identifier |
| `environment` | z. B. `staging` oder `demo` |
| `hostname` | Frontend-Hostname |
| `backendHostname` | Backend-Hostname |
| `frontendImageTag` | unveränderlicher Frontend-SHA |
| `backendImageTag` | unveränderlicher Backend-SHA |
| `databaseSize` | Größe der Tenant-Datenbank |

## Staging vor Demo

Änderungen an Images, Runtime-Konfiguration oder Helm Values werden zuerst in
Staging getestet. Demo wird erst danach auf denselben geprüften SHA gesetzt.

Rollback erfolgt ebenfalls über Git:

1. vorherigen SHA wieder eintragen
2. PR öffnen
3. Staging synchronisieren lassen
4. prüfen
5. bei Erfolg Demo zurücksetzen

## Sicherheitsregeln

- Keine Secret-Werte in Tenant Claims speichern.
- Keine GHCR Tokens in Git speichern.
- Frontend Pull Secret nur über External Secrets bereitstellen.
- AVWX-Key bleibt serverseitig im Backend.
- Secret-Prüfungen nur über Status und Metadaten durchführen.

## Akzeptanzkriterien für einen Tenant

- Namespace existiert.
- TenantApplication ist `SYNCED=True` und `READY=True`.
- Frontend, Backend und Datenbank laufen.
- Frontend und Backend sind über HTTPS erreichbar.
- Zertifikate sind `READY=True`.
- Frontend `config.js` zeigt auf die richtige Backend-URL.
- User, Favorite Location, Forecast und METAR funktionieren.

