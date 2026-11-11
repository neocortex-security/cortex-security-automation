# FAQ

**What do I get from one run?**
A workbook for one asset type and scope: every matching asset, and per asset its
compliance findings and/or vulnerabilities (with packages), plus summary pivots and a
log of every query issued. Run it once per asset type you care about.

**Which asset types?**
Hosts, containers, container images, serverless functions, container services,
container registries, secret/key services, storage services, data services, analytics
and AI services — each as a curated cross-cloud set — plus custom RQL and custom IAM
queries for anything else, and a cloud-account Defender / scan-status report.

**Does it cover CWP (vulnerabilities, packages) as well as CSPM?**
Yes, for hosts, images and functions. A CSPM-only switch skips the package and
vulnerability processing when you just want findings.

**Can I scope it to one account group, one cloud, or a set of tagged assets?**
Yes: cloud type, account group (exact name), asset name substring (optionally
processing only the matches), and asset tag filters.

**Can it report on an asset type that is not in the menu?**
Yes — custom RQL. Any `config from cloud.resource` query; findings are attached to
whatever the query returns. Custom IAM does the same for `config from iam`.

**How big an estate can it handle, and how fast?**
Tens of thousands of hosts with full vulnerability and package enrichment in a single
run; see the README results table for measured figures. Enumeration and enrichment
run as concurrent API flows with per-endpoint ceilings, so the run time is set by the
tenant's rate limits rather than by the tool, and a sequential script doing the same
work takes hours where this takes minutes. Results are held in Polars frames, so the
joins and summaries over millions of rows are not the bottleneck either.

**Can it write somewhere other than a workbook?**
Yes — anywhere Polars can write: local Parquet / CSV / NDJSON / Arrow / Avro, directly
to S3 / Azure Blob / GCS, Delta Lake tables, or a database over SQLAlchemy / ADBC
(PostgreSQL, SQL Server, Snowflake, BigQuery, DuckDB, …). Several targets in one run is
normal: a workbook for the asset owner and Parquet to the data lake for trending.

**Can it run unattended?**
Every prompt has a default and a flag equivalent, so a weekly scheduled run per asset
type is the normal pattern.

**Prisma Cloud is being superseded by Cortex Cloud. Why keep this?**
Two reasons. Until the estate has moved, it is the quickest way to hand asset owners
their findings and vulnerabilities as a table. During the move, it is the Prisma-side
baseline: findings per asset on a known date, and the custom-RQL mode is what rql2xql
uses to measure the Prisma count for a translated rule.

**What permissions does the API key need?**
Read-only: inventory / configuration search, compliance and alerts read, and
vulnerability / Defender read for the CWP data. No write scope is used.

**Where does the data go?**
To files on the machine running the tool. Nothing is sent anywhere but the Prisma
Cloud API. The output contains your estate's findings and software inventory — treat
it accordingly.

**Can my team get it?**
See [ACCESS.md](../ACCESS.md).
