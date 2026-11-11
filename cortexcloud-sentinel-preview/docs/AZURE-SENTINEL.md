# Microsoft Sentinel solution (v7.0)

Python Azure Functions poll the Cortex Cloud public APIs, flatten the nested JSON into
wide custom tables, enrich each record with asset-group membership and deep links, and
ship it to Log Analytics through the Logs Ingestion API. On top of the tables sit KQL
parser functions, analytics rules, hunting queries, Logic App playbooks and 30
workbooks.

## Components

| Component | What it is |
|---|---|
| **Function App 1 — alert & operations** | Four timer functions, every 5 minutes: Issues, Cases, Endpoints, Audit Logs. Per-stream watermark in blob storage; paging stops at 540 s to stay inside the host timeout; the watermark advances only for records actually sent |
| **Function App 2 — asset inventory** | A `TimerDispatcher` writes one queue message per asset class × category (~80); `QueueProcessor` workers drain the queue in parallel, each fetching one class/category at 1 000 records a page, flattening, extracting module fields, resolving asset groups and uploading to that class's table. Daily by default; schedule is an app setting |
| **Ingestion sender** | Shared module: token from the Function App's managed identity, record keys mapped to table columns via a generated column map, upload to the DCE / DCR stream; gzip, 1 MB splitting and `Retry-After` handled |
| **Asset-group cache** | Blob-backed, shared by both apps; gives every record `AssetGroupNames` and `IsPriorityAsset` at ingest time |
| **Infrastructure tooling** | Schema export (snapshots live table schemas), DCR generator, in-place table migration, one deploy script for DCE + DCRs + RBAC + app settings |
| **Content** | 10 KQL parsers (ARM saved searches), 3 analytics rules, 8 hunting queries, 4 playbooks, 30 workbooks — see CONTENT.md |

## Tables (20)

**Alert & operations (every 5 min)**

| Table | Source | Notes |
|---|---|---|
| `CortexCloudIssues_V5_CL` | issue search | default: posture, compute, attack path, … |
| `CortexCloudVulnIssues_V5_CL` | issue search | CVE-centric; vulnerability SLA windows |
| `CortexCloudAppSecIssues_V5_CL` | issue search | IaC, secrets, licence, operational-risk, CI/CD scanners |
| `CortexCloudXDRIssues_V5_CL` | issue search | runtime / endpoint / behavioural |
| `CortexCloudCases_V5_CL` | case search | |
| `CortexCloudEndpoints_V5_CL` | endpoints | incremental by last-seen |
| `CortexCloudAuditLogs_V5_CL` | management audit logs | |

**Issues routing** is by `detection_method`. Every issue record carries `RiskSLADebt`
(days over the remediation window, weighted for runtime and CVE risk; SLA windows per
severity are app settings) plus the fields added in the 2026 API updates (action
status, case ids, exception state and expiry, …).

**Inventory (daily)** — one table per asset class:
Compute · Identity · Network · Data · Security Services · Application · AI · Code ·
API · External Surface · Organization · Management · Other. Each class covers its
categories (Compute alone: VM instance and image, container image / repository /
instance / registry, Kubernetes cluster, serverless function, device, …).

**Schema.** Nested objects and arrays are stringified into top-level columns ("V5"),
sidestepping Log Analytics nesting limits; module configs (DSPM, CIEM, AISPM, CAS, ASM,
API Security) promote their `on_demand` fields into columns. Column names are stable
across the Data Collector → Logs Ingestion migration so parsers, rules and workbooks
did not change.

## Ingestion path

```
Function App (managed identity) ─► Data Collection Endpoint ─► Data Collection Rule
      dcr-cortexcloud-alerts (7 streams)  ·  dcr-cortexcloud-inventory (13 streams)
                                  ─► Log Analytics  Custom-<table>_CL
```

- No workspace shared key anywhere in the solution. The identities need **Monitoring
  Metrics Publisher** on their DCR and **Key Vault Secrets User** on the vault.
- Tables and columns are declared (the Logs Ingestion API does not auto-create them);
  the schema snapshot + DCR generator keep the declaration in step with the code.
- Existing 6.x tables are migrated in place; the two versions can overlap during
  cut-over.

## Closing the loop

Analytics rules raise Sentinel incidents from the tables; automation rules call the
playbooks, which read the Cortex API key from Key Vault with the Logic App's identity
and **write back** — enrich the incident with live asset cards, push status and
assignee to the Cortex case, close the case with resolution. The SOC works in
Sentinel; Cortex stays the system of record.

## Settings that matter

| Setting | Purpose |
|---|---|
| `CORTEX_CLOUD_FQDN`, `CORTEX_CLOUD_API_KEY_ID`, `CORTEX_CLOUD_API_KEY` | tenant and **Advanced** API key (via Key Vault reference) |
| `DCE_ENDPOINT`, `DCR_IMMUTABLE_ID` | ingestion path (set by the deploy script) |
| `InventorySyncScheduleDaily`, `INVENTORY_LOOKBACK_DAYS`, `ASSET_CLASS` | inventory cadence, window, optional worker pinning |
| `RISKSLADEBT_PRIORITY_GROUPS` | asset groups that set `IsPriorityAsset` |
| `RISK_REMEDIATION_*`, `VULN_REMEDIATION_*`, `*_RISK_PENALTY` | SLA windows and debt weighting |

## Prerequisites (summary)

Cortex Cloud tenant with an **Advanced** API key (Issues, Cases R/W, Endpoints, Audit,
Assets, Asset Groups). Azure: Log Analytics + Sentinel, a storage account (queue +
checkpoints), Key Vault, **Basic B1 or higher** App Service plan for the inventory app
(Consumption cannot reliably finish 10-minute inventory runs), two Function Apps on
Python 3.12 / Functions v4.

## Deployment

A step-by-step guide (foundation resources → secrets → Function Apps → ingestion path
→ verify → content), an upgrade procedure from 6.x, a rollback section and a test plan
ship with the solution. Outline: create the API key; create the Azure resources; store
secrets; deploy the two apps; run the ingestion deploy script; confirm tables with
`search "CortexCloud*_V5_CL" | distinct Type`; deploy parsers, workbooks, rules,
playbooks and hunting queries.
