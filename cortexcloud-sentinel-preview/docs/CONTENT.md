# Content catalogue

Everything that sits on top of the tables. Screenshots in `img/` are from the lab
tenant.

## Workbooks (30) — Sentinel

### Persona dashboards (11)

| # | Dashboard | Persona | Data |
|---|---|---|---|
| 01 | Executive Overview | CISO / VP Security | all inventory classes + all four issues tables |
| 02 | All Security | Cloud architects / DevSecOps | all inventory + all issues |
| 03 | Threat Detection & Response | SOC analysts, detection engineers | Compute + Network inventory, XDR issues, attack-path / threat-intel issues |
| 04 | Secure Development | AppSec, DevSecOps | Code + Application + API inventory, AppSec issues |
| 05 | Vulnerability Management | Vulnerability engineers, risk analysts | Vulnerability issues |
| 06 | Identity Security | IAM, cloud security engineers | Identity inventory (CIEM, secrets), issues |
| 07 | Data Security | Data security engineers | Data inventory (DSPM), issues |
| 08 | Compliance | GRC, compliance analysts | Organization + Management + Security Services inventory, CSPM / API-posture issues |
| 09 | AI Security | AI engineers, InfoSec | AI inventory (AISPM), AISPM issues |
| 10 | Attack Surface Management | AppSec, vulnerability management | Compute / Network / Data / External Surface / App / API inventory, attack-path issues |
| 11 | Container & Kubernetes Security | DevSecOps | Compute inventory (K8s, containers), vulnerability + KSPM issues |

### Inventory workbooks (14)

One per asset class — Compute, Identity, Network, Data, Security Services,
Application, AI, Code, API, External Surface, Organization, Management, Other — plus a
summary. Each has global filter pills (subscription, provider, region, account, asset
type), per-tab local filters, charts and a detail table.

### Issues & operations (5)

| Workbook | Content |
|---|---|
| Issues & Cases Trends Tracker | Threats, XDR, runtime compliance, cloud compliance, risks, vulnerabilities, identity, data security, AppSec, cases — trends over time |
| Unified Issues dashboard | 14-tab all-issues view across the four issues tables |
| Security Insights | cross-domain analysis |
| Data Ingestion Monitor | freshness and volume per table; the health check |
| Previous-generation unified dashboard | retained for reference |

## KQL parsers (10)

Workspace functions deployed as ARM saved searches: Issues, Vulnerabilities, AppSec,
Workload Compliance, Identity, DSPM, AISPM, Cases, Endpoints, Audit Logs. Parsers
present a stable query surface over the `_V5_CL` tables so content does not depend on
raw column names.

## Analytics rules (3)

| Rule | Severity | Frequency |
|---|---|---|
| Critical Issue Detected | High | 5 min |
| Multiple Issues on Asset | High | 15 min |
| Case SLA Breach | Medium | 15 min |

## Playbooks (4)

All read the Cortex API key from Key Vault with the Logic App's managed identity.

| Playbook | Direction | Does |
|---|---|---|
| Enrich Issue | Cortex → Sentinel | on incident creation, calls the Cortex API live for each linked asset and writes asset cards into the incident |
| Update Case Status | Sentinel → Cortex | pushes incident status to the Cortex case |
| Assign Case | Sentinel → Cortex | synchronises assignee |
| Close Case | Sentinel → Cortex | closes the Cortex case with resolution |

## Hunting queries (8)

Issue trend analysis · unassigned critical cases · asset risk assessment · stale cases
· issue correlation patterns · analyst workload · recurring issues · stale and offline
endpoints.

## Grafana content — AWS

### Dashboards (30)

One per workbook, same names and personas. Every dashboard carries six global filter
variables backed by OpenSearch terms aggregations — `$priority`, `$provider`,
`$account`, `$region`, `$bizapp`, `$assetgroup` — plus the time picker. Imported
through the Grafana API by a script that ships with the solution (datasource UID is
fixed so the JSON needs no editing).

<!-- Status column: "verified" = checked against live data on the reference deployment; "converted" = JSON ported, pending verification. Update at each release. -->

| Group | Dashboards | Status |
|---|---|---|
| Persona (01–11) | Executive Overview … Container & Kubernetes Security | converted; <<N>> verified |
| Inventory (13 + summary) | one per asset class | converted; <<N>> verified |
| Issues & operations | All Issues, Issues Trends Tracker, Security Insights, Data Ingestion Monitor, v3.5 overview | All Issues and Data Ingestion Monitor verified; rest converted |

### Alert rules (3)

| Rule | Sentinel equivalent | Interval |
|---|---|---|
| Critical issue detected | Critical Issue Detected | 5 min |
| Case SLA breach (critical 4 h / high 24 h / medium 72 h / low 168 h) | Case SLA Breach | 15 min |
| Multiple issues on asset (≥ 3) | Multiple Issues on Asset | 15 min |

### Hunting-query library panels (8)

The eight Sentinel hunting queries as Grafana library panels, droppable into any
dashboard: analyst workload, asset risk assessment, issue correlation patterns, issue
trend analysis, recurring issues, stale cases, stale / offline endpoints, unassigned
critical cases.

### KQL → OpenSearch construct table

| KQL | OpenSearch / Grafana | Caveat |
|---|---|---|
| `summarize count() by X, bin(TimeGenerated, 1h)` | date histogram + terms sub-aggregation | terms size must be explicit |
| `summarize dcount(X)` | cardinality | approximate above the precision threshold |
| `join` across tables | denormalised at ingest, or two panels | the main structural difference |
| `parse` / `extract` | done at ingest, stored as fields | moves work to ingest time |
| `arg_max(TimeGenerated, *)` | top-hits size 1 by `@timestamp` | |
| workbook parameters / filter pills | template variables backed by terms queries | |
| `let` reusable expressions | library panels / variables | |

## Screenshots

### Executive Overview
<!-- TODO add file, then uncomment: ![](img/wb-01-executive-overview.png) -->

### Vulnerability Management
<!-- TODO add file, then uncomment: ![](img/wb-05-vulnerability-management.png) -->

### Compute inventory
<!-- TODO add file, then uncomment: ![](img/wb-inventory-compute.png) -->

### Data Ingestion Monitor
<!-- TODO add file, then uncomment: ![](img/wb-data-ingestion-monitor.png) -->

<!-- one per persona dashboard at minimum; lab tenant, org/subscription names cropped -->
