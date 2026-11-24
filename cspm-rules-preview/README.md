# cspm-rules — a consolidated, validated CSPM rule library for Cortex Cloud

> **Preview repository.** Catalogue, coverage matrix and a handful of complete rules.
> The full library is in the private repository — see
> [Getting the full tooling](#getting-the-full-tooling).

## What this is

A library of Cortex Cloud compliance (posture) rules and correlation rules, in XQL,
each one **validated against a live tenant** and carrying its provenance: where the
control came from, which clouds and asset types it covers, and the measurement that
showed it returns the right population.

It is the consolidated output of several Prisma Cloud → Cortex Cloud migrations: the
custom controls that organisations had written in RQL over years, translated with
[rql2xql](https://github.com/neocortex-security/cortex-security-automation/tree/main/rql2xql-preview), validated, de-duplicated across
estates, and generalised so that they apply to any tenant rather than to the estate
they came from.

## Why it exists

Cortex Cloud ships a large default rule set. What it does not ship is the long tail of
controls that security teams wrote for their own environments — tagging policy,
naming, approved regions, specific logging destinations, encryption-key ownership,
network exposure patterns particular to a platform team's design. Those are the rules
that take the longest to rebuild after a migration and the ones most often lost. This
library is that tail, generalised.

## Catalogue

Counts as of November 2026: the rules in the library that deploy on a Cortex Cloud 2.3 tenant
(rules a Cortex default already covers are not counted).

| Cloud | Posture rules | Correlation rules | Asset types covered | Last validated |
|---|---|---|---|---|
| AWS | 512 | 0 | 350 | 2026-10 |
| Azure | 299 | 0 | 239 | 2026-10 |
| GCP | 215 | 0 | 98 | 2026-10 |
| OCI | 17 | 0 | 17 | 2026-10 |
| Alibaba | 1 | 0 | 1 | 2026-10 |
| Kubernetes | — | — | — | — |
| **Total** | **1,044** | **0** | **701** | |

Of the 1,044, 88 deploy cleanly but could not be checked against a population, because the
validation tenants hold no assets of that type (AWS 28, Azure 25, GCP 20, OCI 14, Alibaba 1).
Kubernetes-API checks and correlation rules are not part of this release.

Compared with the posture rules Cortex Cloud ships by default (994 on a 2.3 tenant,
exported through the rule API in November 2026):

| Cloud | Library rules | Cortex default rules | Asset types, library | Asset types, defaults | Library asset types with no default rule |
|---|---|---|---|---|---|
| AWS | 512 | 382 | 350 | 112 | 256 |
| Azure | 299 | 285 | 239 | 64 | 185 |
| GCP | 215 | 213 | 98 | 45 | 69 |
| OCI | 17 | 66 | 17 | 23 | 10 |
| Alibaba | 1 | 48 | 1 | 11 | 0 |
| **Total** | **1,044** | **994** | **701** | **255** | **515** |

Severity, library vs defaults: High 153 vs 106, Medium 455 vs 405, Low 436 vs 177,
Informational 0 vs 306.

By control family — see [docs/CATALOGUE.md](docs/CATALOGUE.md) for the full list with
one line per rule (name, cloud, asset type, severity, framework mappings, status).

| Family | Rules |
|---|---|
| Logging & monitoring | 110 |
| Encryption & key management | 167 |
| Network exposure | 194 |
| Identity & access | 156 |
| Tagging, naming, governance | 142 |
| Backup & resilience | 47 |
| Data services (DB, storage, queues) | 82 |
| Compute & containers | 65 |
| AI services | 81 |

## What "validated" means

Every rule in the library carries:

- the XQL, in the form the Cortex rule API accepts for that asset type;
- the asset types and raw-field paths it reads, checked against collected data (not
  documentation) for casing and shape;
- the control intent in one sentence, and the source (custom control generalised /
  gap in default coverage / framework requirement);
- the measurement: tenant class, date, population returned, and the comparison it was
  validated against;
- framework mappings where they exist (CIS, NIST, ISO 27001, PCI, …).

Rules whose check cannot be fully expressed in Cortex Cloud (because a field is not
collected) are **in the library marked as partial**, with the gap stated, rather than
silently omitted — see the FAQ on coverage gaps.

## Sample rules

`docs/SAMPLE_RULES/` *(coming soon)* will contain <<5–8>> complete entries in the
library's format, one per cloud, chosen to show the range (a simple scalar check, a
list check, a join, a correlation rule, a partial).

## Distribution

The library is delivered as a workbook + JSON set that
[cspm-rules-manager](https://github.com/neocortex-security/cortex-security-automation/tree/main/cspm-rules-manager-preview) `cortex
create` deploys directly, so a tenant goes from default rules to default + library in
one reviewed run. Updates are versioned; a changelog lists rules added, changed and
withdrawn per release.

## What is in this repository

| Path | Contents |
|---|---|
| [docs/CATALOGUE.md](docs/CATALOGUE.md) | One line per rule, no XQL |
| `docs/SAMPLE_RULES/` *(coming soon)* | Complete entries in library format |
| [docs/FORMAT.md](docs/FORMAT.md) | The entry schema |
| [docs/FAQ.md](docs/FAQ.md) | Provenance, coverage gaps, licensing |
| [ACCESS.md](../ACCESS.md) | How to get the full library |


---

## Getting the full tooling

This tooling was built in the course of Palo Alto Networks Professional Services
engagements and is made available to organisations working with me on a Cortex Cloud
deployment or migration. See [ACCESS.md](../ACCESS.md).

## Related previews

| Repo | What it is |
|---|---|
| [rql2xql-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/rql2xql-preview) | Prisma Cloud RQL → Cortex Cloud XQL translator with live validation |
| [cspm-rules-manager-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cspm-rules-manager-preview) | One CLI for Prisma policies, Cortex posture and correlation rules, allowlists, exceptions and AppSec rules |
| [cspm-rules-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cspm-rules-preview) | Consolidated, validated CSPM rule library for Cortex Cloud |
| [cortex-cloud-terraform-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cortex-cloud-terraform-preview) | Provisions the managed security service framework for a Cortex Cloud tenant, as code |
| [prisma-asset-reporter-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/prisma-asset-reporter-preview) | Per-asset compliance findings and vulnerabilities from Prisma Cloud, cross-cloud, by asset type or custom RQL |
| [cortexcloud-sentinel-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cortexcloud-sentinel-preview) | Cortex Cloud in your SIEM: Microsoft Sentinel solution and its AWS OpenSearch + Grafana counterpart |

---

*Palo Alto Networks, Prisma Cloud, Cortex, Cortex Cloud, Cortex XDR and Cortex XSIAM are
trademarks of Palo Alto Networks, Inc. This is not an official Palo Alto Networks project.
Figures quoted describe specific engagements at the dates given.*
