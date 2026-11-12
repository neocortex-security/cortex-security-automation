# prisma_asset_reporter — per-asset compliance findings and vulnerabilities from Prisma Cloud, across every cloud

> **Preview repository.** Report catalogue, asset-type coverage, architecture and sample
> output. Source is in the private repository — see
> [Getting the full tooling](#getting-the-full-tooling).

<!-- TODO add file, then uncomment: ![demo](docs/demo.gif) -->

## The problem

Prisma Cloud knows, for every asset in an estate, what it is, which compliance checks
it fails, and which vulnerabilities it carries — but it presents that per console view,
per cloud, per module (CSPM findings in one place, CWP vulnerabilities and runtime
packages in another), and never as one table you can hand to an asset owner, a risk
team or a migration project. Exporting it by hand means stitching several APIs with
different paging, filters and identifiers.

## What prisma_asset_reporter does

One interactive (or scripted) run produces, for a chosen **asset type** and scope,
a workbook with every matching asset and — per asset — its **compliance findings**,
its **vulnerabilities** (with runtime package detail where the agent/scanner provides
it), or both.

**Scope selectors**

| Selector | Values |
|---|---|
| Cloud type | AWS · Azure · GCP · OCI · Alibaba · IBM · other / on-prem · **all** |
| Account group | exact Prisma Cloud account-group name, or no filter |
| Asset name filter | substring, optionally *process only the filtered assets* |
| Asset tags | tag key/value filters |
| Process type | findings · vulnerabilities · inventory only |
| Runtime packages | include, or skip for a CSPM-only run |

**Asset types**

| # | Asset type | Findings | Vulnerabilities |
|---|---|---|---|
| 1 | Hosts (VMs, bare metal, on-prem) | ✅ | ✅ |
| 2 | Containers (running) | ✅ | |
| 3 | Container images | ✅ | ✅ |
| 4 | Serverless functions | ✅ | ✅ |
| 5 | Container services (EKS / AKS / GKE / ECS / …) | ✅ | |
| 6 | Container registries | ✅ | |
| 7 | Secret / key services (KMS, Key Vault, Secrets Manager, …) | ✅ | |
| 8 | Storage services (object, block, file) | ✅ | |
| 9 | Data services (databases, warehouses, caches, queues) | ✅ | |
| 10 | Analytics and AI services | ✅ | |
| 11 | **Custom RQL** — any `config from cloud.resource` query, any asset | ✅ | |
| 12 | **Custom IAM** — any `config from iam` query | ✅ | |
| 13 | Cloud accounts — Defender / agent coverage and scan status | | |

Types 1–10 are curated query sets per cloud, so "Storage services for all clouds"
means S3 + Azure Storage + GCS + OCI Object Storage + OSS + … in one run, with the
findings normalised to one column set. Types 11–12 take your own RQL, so there is no
asset the tool cannot report on.

## Example run

```
$ prisma_asset_reporter
--- Configuring Processing Run ---
Cloud Type: aws/azure/gcp/oci/alibaba/ibm/other(on-prem) (default: 'all'):
ACCOUNT GROUP name (exact match) - e.g., 'test', default: '' (No filter):

Select the asset type for all :
  1. Hosts (findings, vulnerabilities)
  2. Containers (findings)
  3. Container Images (findings, vulnerabilities)
  4. Functions (findings, vulnerabilities)
  5. Container Services (findings)
  6. Container Registries (findings)
  7. Secret/Key Services (findings)
  8. Storage Services (findings)
  9. Data Services (findings)
  10. Analytics and AI Services (findings)
  11. Custom RQL (1 queries) (findings)
  12. Custom IAM (1 queries) (findings)
  13. Cloud Accounts Defence/Scan Status
Enter your choice (default: '1'): 1
Process compliance findings or vulnerabilities? (f/v/n) (default: 'n'): v
Skip runtime package/vulnerability processing (CSPM only)? (y/n) (default: 'n'):
ASSET NAME filter string - e.g., 'test', default: '' (No filter):
Process ONLY the filtered assets? (y/n) (default: 'n'):
Filter by ASSET TAGS? (y/n) (default: 'n'):

--- Configuration Summary ---
  cloud_type: all
  account_group:
  asset_type: host
  process_type: vulnerabilities
  cspm_only: False
  process_package: True
  asset_type_stage: all
  asset_type_runc: False
  filter_str:
  filter_only: False
  asset_tags_filters: []
---------------------------

--- Login Successful ---
  Config: COMPANY=<alias> -> API_config-<alias>.ini (https://api<n>.prismacloud.io)
  Customer Name: <tenant display name> - <id>
  Prisma ID: <id>
------------------------
```

Every prompt has a default, and the same choices can be supplied non-interactively
for scheduled runs.

## Built for large estates

The enrichment step is where naive scripts die: one API call per asset for findings,
another for vulnerabilities, another for packages, across tens of thousands of assets.
prisma_asset_reporter runs those as **concurrent API flows** — asset enumeration,
findings and vulnerability enrichment pipelined against each other with per-endpoint
concurrency ceilings and back-off — so a run is bounded by the tenant's rate limits,
not by the tool. Results are assembled in **Polars** dataframes, so joins, pivots and
severity roll-ups over millions of rows happen in memory in seconds and the output
step is a single write.

**Export to wherever the data needs to go.** Because the result set is a Polars frame,
it can be written to any destination Polars supports, not just a workbook:

| Destination | Formats |
|---|---|
| Local files | XLSX, CSV, JSON / NDJSON, Parquet, Arrow IPC, Avro |
| Object storage | S3, Azure Blob / ADLS, GCS — Parquet / CSV / NDJSON written directly to the bucket |
| Lakehouse | Delta Lake tables |
| Databases | any SQLAlchemy / ADBC target — PostgreSQL, SQL Server, Snowflake, BigQuery, DuckDB, SQLite … |
| Downstream analysis | Arrow interchange to pandas, DuckDB, Spark without a copy |

So the same run can hand an asset owner a workbook and land the full dataset as
Parquet in a security data lake, partitioned by cloud and date, for trending.

## Results

<!-- Dated, no client names. Put the biggest run first. -->

| Estate | Asset type | Assets | Enrichment calls | Output rows | Run time |
|---|---|---|---|---|---|
| <<sector>>, 4 clouds | Hosts, vulnerabilities + packages | <<N>> k | <<N>> k | <<N>> M | <<N>> min |
| <<sector>> | Storage services, findings, all clouds | <<N>> k | <<N>> k | <<N>> k | <<N>> min |
| <<sector>> | Custom RQL, <<N>> queries | <<N>> k | <<N>> k | <<N>> k | <<N>> min |

Sequential equivalent of the first row at the same rate limit: <<N>> hours.

## Where it fits

- **Asset-owner reporting** — "every host in account group X with critical
  vulnerabilities and the packages behind them", as a workbook, weekly.
- **Migration baseline** — the Prisma-side record of findings per asset on the day of
  cut-over, for comparison against Cortex Cloud. The custom-RQL and custom-IAM modes
  are what [rql2xql](https://github.com/neocortex-security/cortex-security-automation/tree/main/rql2xql-preview) uses to measure the
  Prisma side of a translation.
- **Coverage audit** — type 13 answers "which cloud accounts are actually defended and
  scanned", which is the first question in any CWP conversation.

## What is in this repository

| Path | Contents |
|---|---|
| [docs/REPORTS.md](docs/REPORTS.md) | Output workbook per asset type: sheets, columns, what each is joined from |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Query sets per cloud, finding/vulnerability enrichment, paging |
| `docs/SAMPLE_OUTPUT/` *(coming soon)* | Sanitised workbooks: one hosts/vulnerabilities, one storage/findings, one account status |
| [docs/FAQ.md](docs/FAQ.md) | Scope, limits, data handling |
| [ACCESS.md](../ACCESS.md) | How to get the tool |

## What is **not** in this repository

Source; the curated per-cloud query sets behind asset types 1–10; and the request
shapes for the IAM and vulnerability/package endpoints, which differ from the
documented configuration-search API and are most of what the tool knows.


---

## Getting the full tooling

This tooling was built in the course of Palo Alto Networks Professional Services
engagements and is made available to organisations working with me on a Cortex Cloud
deployment or migration. See [ACCESS.md](../ACCESS.md).

## Related previews

| Repo | What it is |
|---|---|
| [rql2xql-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/rql2xql-preview) | Prisma Cloud RQL → Cortex Cloud XQL translator with live validation |
| [cspm-rules-manager-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cspm-rules-manager-preview) | One CLI to collect, update and create Prisma policies, Cortex rules and AppSec rules |
| [cspm-rules-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cspm-rules-preview) | Consolidated, validated CSPM rule library for Cortex Cloud |
| [cortex-cloud-terraform-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cortex-cloud-terraform-preview) | Provisions the managed security service framework for a Cortex Cloud tenant, as code |
| [prisma-asset-reporter-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/prisma-asset-reporter-preview) | Per-asset compliance findings and vulnerabilities from Prisma Cloud, cross-cloud, by asset type or custom RQL |
| [cortexcloud-sentinel-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cortexcloud-sentinel-preview) | Microsoft Sentinel → AWS (OpenSearch + Grafana) data pipeline |

---

*Palo Alto Networks, Prisma Cloud, Cortex, Cortex Cloud, Cortex XDR and Cortex XSIAM are
trademarks of Palo Alto Networks, Inc. This is not an official Palo Alto Networks project.
Figures quoted describe specific engagements at the dates given.*
