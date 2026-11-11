# Operational notes

Findings from building and running the integration on both platforms that are not in
the vendor documentation, or are there but easy to miss. Published because they are
generically useful; the solution-specific handling stays private.

## Azure — ingestion

- **The HTTP Data Collector API is retired.** Microsoft retired the `SharedKey`
  HMAC API on 14 September 2026 — not a hard cut-off, but unsupported. Anything still
  posting to `*.ods.opinsights.azure.com` should move to the Logs Ingestion API (DCE +
  DCR). The solution's 7.0 release is that move; existing tables are migrated in place
  and content is unchanged.
- **The Logs Ingestion API does not auto-create tables or columns.** A column the code
  sends that the DCR stream does not declare is silently dropped — no error. Snapshot
  the live table schemas, generate the DCRs from the snapshots, and treat "add a field"
  as a schema change (table + schema file + column map + DCR).
- **Table migration to DCR-based is irreversible**, but the old sender keeps working on
  migrated tables, so the two versions can overlap during cut-over.
- **DCR limits**: 2 GB/min and 12 000 requests/min per DCR; the SDK honours
  `Retry-After`. A `403` from the DCE is almost always a missing **Monitoring Metrics
  Publisher** on the DCR (allow ~10 min after assignment) or a private-link / VNet
  restriction.
- **Column naming**: the inferred suffixes from the old API (`_s`, `_d`, `_b`, `_t`,
  `_g`) are kept by mapping base field names to column names, so parsers and workbooks
  survive the migration untouched.

## Azure — Functions

- **Consumption plans cannot reliably finish 10-minute inventory runs.** Basic B1 or
  higher for the inventory app. The alert collectors stop paging at 540 s and the
  inventory workers at 570 s; both upload what they have and advance the watermark
  only for records actually sent.
- **Fan the inventory out over a queue**, one message per asset class × category.
  ~80 small executions in parallel finish; one big execution does not. `batchSize` and
  a per-page sleep bound parallelism under the Cortex rate limit.
- **Subscription policies that force `publicNetworkAccess = Disabled` on Function
  Apps break the integration** (no outbound to Cortex, no outbound to the DCE, no
  `func` deploy). The solution ships a compliant template (VNet integration, service
  endpoints, Key Vault) and a policy-exemption request template for the cases where
  the policy cannot be met.
- **Remote build** (`SCM_DO_BUILD_DURING_DEPLOYMENT`, `ENABLE_ORYX_BUILD`) or
  `ModuleNotFoundError` on first run.
- **KQL booleans**: columns stringified in the V5 schema must not be `tobool()`-ed;
  compare lower-cased strings.

## Cortex Cloud API

- The tenant FQDN is used **without** the `api-` prefix for deep links and **with** it
  for calls; most `401`s are a prefix, scheme or trailing-slash problem, or a Standard
  key where an **Advanced** one is required.
- Issues carry a `detection_method` that is the right routing key; routing into
  vulnerability / AppSec / XDR / general tables gives each its own field set and SLA
  model and keeps the general table queryable.
- The 2026 API updates added fields and changed response wrappers on issue and case
  search; the solution tracks them in a change log with validation queries.
- Asset-group membership is not on the issue or asset record; resolve it at ingest
  from a cached group listing and stamp `AssetGroupNames` / `IsPriorityAsset`.

## AWS — Lambda and state

- **Lambda's 15-minute ceiling is not the constraint; OpenSearch and the Cortex rate
  limit are.** The inventory is fanned out over SQS (79 messages, batch size 1,
  reserved concurrency 10) with a 570 s safety stop per worker and a DLQ after three
  retries; the scheduled ingestors stop at 540 s. Same shape as the Azure queue
  design, which is why the two stay in step.
- **Watermarks need a monotonic conditional write.** Two overlapping invocations
  (a retry, a manual trigger) must not move the watermark backwards; DynamoDB's
  conditional `PutItem` does it in one call. Blob storage on Azure needed the same
  discipline by other means.
- **Secrets are created with placeholders by the stacks** and must be set after the
  first deploy; warm containers cache the value, so a rotation takes effect as
  containers recycle (new containers always fetch fresh).

## AWS — OpenSearch

- **FGAC role mapping distinguishes users from roles.** IAM *users* go in `users`; IAM
  *roles* go in `backend_roles`. A role ARN in `users` produces a 403 with no useful
  message. The Lambda execution roles and the Grafana workspace role all have to be
  mapped; the guide does it through a signed local proxy to OpenSearch Dashboards.
- **Master user ARN** must be set at domain creation (or recovered afterwards with a
  documented procedure); getting it wrong is the common cause of a deploy that
  succeeds and a pipeline that writes nothing.
- **IAM policy resource scope**: the pipeline needs `es:ESHttp*` on
  `arn:aws:es:<region>:<account>:domain/<name>/*` — the `/*` matters.
- **Index templates before first write**, or dynamic mapping chooses wrong types and
  the index must be reindexed.

## AWS — Grafana

- **API keys were removed in Grafana 12.4.** Provisioning must use service accounts
  and tokens. Amazon Managed Grafana follows upstream with a lag.
- **The OpenSearch datasource needs "Get Version and Save"** once before it works;
  file provisioning alone leaves it non-functional.
- **Datasource UID is referenced by every dashboard JSON.** Fix the UID at creation
  (the solution uses one constant) or every import needs editing.
- **Terms aggregation size** must be explicit on panels converted from KQL
  `summarize … by X`; the default truncates silently.
- **A datasource plugin change can break every converted dashboard at once** (query
  model differences between the Elasticsearch-compatible and the OpenSearch
  datasources). Keep the conversion scripted so the whole set can be regenerated.
