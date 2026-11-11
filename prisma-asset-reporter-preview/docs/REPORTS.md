# Report catalogue

One run → one workbook (plus CSV/JSON of the same data). The sheets depend on the asset
type and process type chosen. Column lists are abbreviated; the samples under
`SAMPLE_OUTPUT/` show the full header.

<!-- Keep in sync with the writers. Generate the column lists if possible. -->

## Output naming

```
<asset-type>_<process-type>_<cloud>_<account-group|all>_<YYYYMMDD-HHMM>.xlsx
e.g. host_vulnerabilities_all_all_20261104-2310.xlsx
```

## Sheets

### `Assets` — always present

One row per asset in scope.

| Column | From |
|---|---|
| asset id (Prisma RRN / unified id) | inventory |
| cloud, account id, account name, account group(s) | inventory |
| region | inventory |
| asset type (Prisma API name), service | query set |
| name, tags | inventory |
| created, last modified / last scanned | inventory |
| Defender / agent status (hosts, containers, functions) | CWP |
| findings count by severity | derived from `Findings` |
| vulnerabilities count by severity | derived from `Vulnerabilities` |
| highest severity | derived |
| console link | built |

### `Findings` — process type *findings*

One row per (asset, failing policy).

| Column | From |
|---|---|
| asset id, name, cloud, account | join |
| policy id, policy name, policy type | compliance / policy |
| severity | policy |
| compliance standards → requirements → sections | policy mappings |
| finding status, first seen, last seen | alerts / posture |
| remediation available | policy |
| alert id (if an alert exists) | alerts |

### `Vulnerabilities` — process type *vulnerabilities* (hosts, images, functions)

One row per (asset, CVE, package).

| Column | From |
|---|---|
| asset id, name, cloud, account | join |
| CVE, CVSS, severity, vendor severity | CWP vulnerability |
| package name, installed version, fixed version, package type | runtime packages (skipped in CSPM-only mode) |
| exploit available, in-use / running (where the agent reports it) | CWP |
| published, discovered | CWP |
| risk factors | CWP |

### `Packages` — when runtime packages are processed

One row per (asset, package), independent of CVEs — the software inventory.

### `Summary`

Pivots: assets × cloud × account; findings by severity × standard; vulnerabilities by
severity × fix-availability; top CVEs by asset count; top packages.

### `Account_Status` — asset type 13

One row per cloud account: cloud, account id, name, group(s), onboarding status,
CSPM scan status / last scan, agentless scan status, Defender coverage (hosts with /
without), last ingest, errors.

### `Query_Log`

Every query the run issued (RQL text or endpoint), its scope, row count and duration.
This is the audit trail for the run and the input to rql2xql's Prisma-side measurement.

## Custom RQL / custom IAM

Queries come from a file (`queries/custom_rql.txt`, `queries/custom_iam.txt`), one per
line with an optional `name|` prefix. Each query produces its own block in `Assets`
and `Findings`, tagged with the query name; the `Query_Log` records the exact text.
IAM queries use the IAM-specific search endpoint and return principal → permission →
resource rows rather than assets.

### `Errors`

Assets that could not be enriched after retries: asset id, stage, HTTP status, message.
Empty on a clean run; its presence means the run is complete but not total.

## Formats and destinations

The sheets above are Polars frames; each can be written to one or more targets per
run.

| Target | Flag | Notes |
|---|---|---|
| XLSX | default | one sheet per frame, frozen headers, filters, severity colouring |
| CSV / NDJSON / JSON | `--out csv\|ndjson\|json` | one file per frame; JSON nests findings / vulnerabilities / packages under each asset |
| Parquet / Arrow IPC / Avro | `--out parquet\|ipc\|avro` | `--partition cloud,account,date` for Parquet |
| S3 / Azure Blob / GCS | `--out s3://… \| az://… \| gs://…` | Parquet / CSV / NDJSON written directly to the bucket |
| Delta Lake | `--out delta://…` | append (default) or `--overwrite` |
| Database | `--out <sqlalchemy-or-adbc-uri>` | one table per frame, `--table-prefix` |

Multiple `--out` flags are allowed; the frames are built once and written to each.
