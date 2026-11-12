# rql2xql — Prisma Cloud RQL → Cortex Cloud XQL, validated against live tenants

> **Preview repository.** Documentation, methodology, worked examples and measured
> results. Source, schema catalogue and override library are in the private repository —
> see [Getting the full tooling](#getting-the-full-tooling).

<!-- TODO add file, then uncomment: ![demo](docs/demo.gif) -->

## The problem

A Prisma Cloud estate carries hundreds to thousands of custom CSPM policies written in
RQL. Cortex Cloud expresses the same checks as XQL over `asset_inventory` (and, for IAM,
audit-event and network policies, over the identity, `cloud_audit_logs` and flow
datasets). The two languages differ in list semantics, null handling, join behaviour,
field casing and path layout, so a syntactically plausible translation is routinely
**wrong** — it runs, returns a number, and the number is not what the RQL returns.

Count-only QA ("within 5 % of Prisma") does not catch this: a rule can match on count
and miss on population.

## What rql2xql does

1. **Translates** each RQL policy to XQL using a schema-aware path resolver over
   <<N>>+ asset types and <<N>> raw-field paths, with a curated override layer for the
   patterns the engine cannot derive.
2. **Runs both sides live** — the RQL on the Prisma tenant, the XQL on the Cortex tenant
   (prod and dev in one run) — and compares populations, not just counts.
3. **Classifies every disagreement** into a named anomaly class (collection gap, path
   casing, list-vs-scalar read, join fan-out, null arm, …) so the fix is targeted.
4. **Emits a review workbook** per batch: translation, both counts, % diff, anomaly
   class, override provenance, and a deployable rule definition.
5. **Deploys** validated rules to Cortex Cloud as posture rules or correlation rules
   according to the policy type.

## Results


Across 5K distinct policies to date. Full definitions of
each column are in [docs/METHODOLOGY.md](docs/METHODOLOGY.md).

## Scope

| Prisma policy type | Source dataset in Cortex | Cortex rule target | Status |
|---|---|---|---|
| `config from cloud.resource` | `asset_inventory` | posture rule (advanced XQL) | production |
| `config from iam` | identity-security datasets | correlation rule | <<status>> |
| `event from cloud.audit_logs` | `cloud_audit_logs` | correlation rule | <<status>> |
| `network from vpc.flow_record` | `network_story` / raw flow logs | correlation rule | <<status>> |

Clouds: AWS, Azure, GCP, OCI, Alibaba (as present in the source policies).
Also carries Prisma **alert exceptions** (dismissed / snoozed) over to Cortex issue
exceptions.

## What is in this repository

| Path | Contents |
|---|---|
| [docs/METHODOLOGY.md](docs/METHODOLOGY.md) | The translate → measure → classify → override loop, and what "validated" means |
| [docs/EXAMPLES.md](docs/EXAMPLES.md) | Worked RQL → XQL examples that show the non-obvious transformations |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Components and data flow |
| `docs/SAMPLE_OUTPUT/` *(coming soon)* | A sanitised review workbook and anomaly report |
| [docs/FAQ.md](docs/FAQ.md) | Scope, limits, how it compares with count-only validation |
| [CHANGELOG.md](CHANGELOG.md) | Release history (headline items) |
| [ACCESS.md](../ACCESS.md) | How to get the full tooling |

## What is **not** in this repository

The translator, the asset schema catalogue, the override library, the anomaly
rules ledger and the manual-override Claude skill.

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
