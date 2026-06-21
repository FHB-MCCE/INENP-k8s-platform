# Kostenstand und Abweichungsanalyse

Dieses Dokument ergänzt das Cost-Planning-Issue und hält fest, wie der
Ist-Kostenstand für Gate 7 dokumentiert wird.

## Geplantes Kostenmodell

Die Plattform nutzt eine kostenbewusste GKE-Standard-Umgebung:

| Ressource | Planung |
|---|---|
| Cluster | GKE Standard, zonal |
| Region/Zone | `europe-west3-a` |
| Node Pool | 3 bis 4 Nodes |
| Node Type | `n2-standard-2` |
| Datenbank | CloudNativePG im Cluster |
| DNS | Cloud DNS |
| Registry | GHCR |
| Secrets | Google Secret Manager |

## Ist-Kosten erfassen

Vor der Abgabe:

1. GCP Billing öffnen.
2. Projekt `dulcet-velocity-495612-j0` filtern.
3. Zeitraum vom ersten Apply bis zum Abgabedatum wählen.
4. Screenshot mit Datum und Gesamtkosten im Cost-Planning-Issue anhängen.
5. Größte Kostentreiber kurz notieren.

Keine Billing-Exports mit personenbezogenen oder Zahlungsdaten veröffentlichen.

## Erwartete Abweichungen

| Abweichung | Ursache |
|---|---|
| Niedrigere Kosten als Monatsplanung | Cluster lief nur während Projektphase, nicht einen vollen Monat. |
| GKE Compute dominiert | Drei `n2-standard-2` Nodes sind der größte laufende Kostenblock. |
| DNS und Secret Manager gering | Wenige Records und wenige Secret-Versionen. |
| Keine Cloud SQL Kosten | Datenbank läuft über CloudNativePG im vorhandenen Cluster. |
| Kurzfristige Mehrkosten möglich | E2E-Tests, LoadBalancer und mehrfaches Apply/Sync während Stabilisierung. |

## Gate-7 Nachweis

Für Gate 7 muss im Issue stehen:

- Datum des Kostenstands
- Screenshot oder belastbarer Billing-Auszug
- kurzer Vergleich mit der ursprünglichen Planung
- Erklärung der wichtigsten Abweichungen
- Entscheidung, ob Ressourcen nach der Demo abgebaut werden

## Abbau-Hinweis

Vor einem späteren Terraform Destroy zuerst sichern:

- Screenshots der Kosten
- Argo CD Status
- TenantApplication Status
- HTTPS Smoke-Test-Ergebnisse
- Präsentations- und Demo-Nachweise

