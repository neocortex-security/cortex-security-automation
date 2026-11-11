# Cost profile

Indicative, to set expectations. Your numbers will differ with estate size, retention
and region.

## What drives volume

| Stream | Daily records | Notes |
|---|---|---|
| Issues (4 tables) | 100 – 1 000 | incremental by watermark |
| Cases | 10 – 100 | incremental |
| Endpoints | 100 – 10 000 | incremental by last-seen |
| Audit logs | 500 – 5 000 | incremental |
| Inventory (13 tables) | 10 000 – 200 000 assets | full daily snapshot of assets seen within the lookback window |

Inventory dominates. Shortening `INVENTORY_LOOKBACK_DAYS` or pinning classes reduces
it; so does excluding categories nobody dashboards.

## Microsoft Sentinel

| Estate | Ingestion / month | Log Analytics + Sentinel (<<region>>, <<YYYY-MM>> list) | Functions (B1 plan ×1, Consumption ×1), storage, Key Vault | Total |
|---|---|---|---|---|
| Small (≤ 10 k assets) | ~1 – 3 GB | <<$>> | <<$>> | **<<$>>** |
| Medium (10 – 50 k) | ~5 – 10 GB | <<$>> | <<$>> | **<<$>>** |
| Large (50 – 200 k) | ~15 – 25 GB | <<$>> | <<$>> | **<<$>>** |

Log Analytics ingestion is billed per GB (with Sentinel's per-GB charge on top unless
on a commitment tier); the Basic B1 plan for the inventory app is a flat monthly cost;
everything else is noise. Interactive retention beyond the included period is the
other lever.

## AWS (OpenSearch + Grafana)

| Estate | OpenSearch (nodes + storage) | Lambda + SQS + DynamoDB + Secrets | Managed Grafana | Total |
|---|---|---|---|---|
| Small (≤ 10 k assets) | 1 × `r6g.large.search`, 100 GiB gp3 — ~$150–200 | <<$>> (tens) | per active user | **<<$>>** |
| Medium (10 – 50 k) | <<nodes>> — <<$>> | <<$>> | per active user | **<<$>>** |
| Large (50 – 200 k) | <<nodes>> — <<$>> | <<$>> | per active user | **<<$>>** |

OpenSearch is 70 – 85 % of the AWS bill at every tier; node count and storage drive
it. Lambda, SQS and DynamoDB (on-demand) are rarely material at these volumes.
Amazon Managed Grafana bills per active editor / viewer. Reserved instances take
30 – 50 % off the dominant line.

## Comparison worth making

Against the cost of **not** having the data in the SIEM: analysts pivoting to a second
console, compliance reporting assembled by hand, asset owners without a view of their
own findings. And against a vendor connector that carries alerts only — the inventory
and module fields are the part this integration adds, and the part that costs the GB.
