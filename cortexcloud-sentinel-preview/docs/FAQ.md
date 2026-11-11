# FAQ

**Is this the vendor's Sentinel connector?**
No. The vendor connectors carry alerts. This solution carries issues (routed into four
tables with their own field sets), cases, endpoints, audit logs **and the full asset
inventory** across 13 classes, with module-specific fields (DSPM, CIEM, AISPM, CAS,
ASM, API Security), asset-group context and deep links — plus 30 workbooks, parsers,
rules, hunting queries and bidirectional case playbooks.

**Why both Azure and AWS?**
Because the customer's security data platform decides. Sentinel customers get the
Function App solution; AWS-centric customers get the same collectors, data model and
dashboards on Lambda, OpenSearch and Grafana. A customer with both sees the same
tables and dashboards on each side.

**Which is more mature?**
Sentinel: v7.0, production, nine releases since October 2025, with bidirectional case
playbooks. AWS: v1.0 (June 2026) at parity with the Sentinel v6.0 data model — all 20
indices populated, all 30 dashboards converted, live verification of the dashboards
in progress; case write-back has no AWS equivalent yet.

**Is the AWS stack serverless?**
Apart from the OpenSearch domain and the Grafana workspace, yes: Lambda, EventBridge,
SQS, DynamoDB on-demand, Secrets Manager. One CDK app, four stacks, no servers to
patch.

**Does it need a workspace key / shared key?**
No. Version 7.0 uses the Logs Ingestion API with managed identity (DCE + DCR). The
HTTP Data Collector API that needed the shared key was retired by Microsoft in
September 2026; 6.x deployments upgrade in six scripted steps with content unchanged.

**How fresh is the data?**
Alert streams every 5 minutes with watermarks; inventory daily (configurable), as a
snapshot of assets seen within the lookback window.

**What does the inventory run cost in Functions terms?**
It is fanned out over a storage queue — ~80 tasks per run — so no single execution
approaches the 10-minute timeout. The inventory app needs a Basic B1 plan or higher;
the alert app runs on Consumption.

**Can the SOC act on a Cortex case from Sentinel?**
Yes. Playbooks push status, assignee and closure back to the Cortex case, and enrich
the Sentinel incident with live asset cards from the Cortex API. Cortex stays the
system of record.

**Our subscription policy forces `publicNetworkAccess = Disabled` on Function Apps.**
Then the functions cannot reach Cortex or the DCE. The solution ships a compliant
template (VNet integration, service endpoints, Key Vault) and a policy-exemption
request template for when the policy cannot be met. See NOTES.md.

**Do KQL workbooks convert to Grafana automatically?**
No. Each is ported by hand to OpenSearch aggregations; the recurring constructs are
tabulated in CONTENT.md. Joins are resolved at ingest.

**What permissions does the Cortex API key need?**
An **Advanced** key with Issues read, Cases read/write, Endpoints read, Audit Logs
read, Assets read, Asset Groups read.

**Can my team get it?**
See [ACCESS.md](../ACCESS.md).
