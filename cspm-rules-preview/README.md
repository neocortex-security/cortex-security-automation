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

<!-- Fill from the private library. Counts dated. -->

| Cloud | Posture rules | Correlation rules | Asset types covered | Last validated |
|---|---|---|---|---|
| AWS | <<N>> | <<N>> | <<N>> | <<YYYY-MM>> |
| Azure | <<N>> | <<N>> | <<N>> | <<YYYY-MM>> |
| GCP | <<N>> | <<N>> | <<N>> | <<YYYY-MM>> |
| OCI | <<N>> | <<N>> | <<N>> | <<YYYY-MM>> |
| Alibaba | <<N>> | <<N>> | <<N>> | <<YYYY-MM>> |
| Kubernetes | <<N>> | <<N>> | <<N>> | <<YYYY-MM>> |
| **Total** | **<<N>>** | **<<N>>** | **<<N>>** | |

By control family — see [docs/CATALOGUE.md](docs/CATALOGUE.md) for the full list with
one line per rule (name, cloud, asset type, severity, framework mappings, status).

| Family | Rules |
|---|---|
| Logging & monitoring | <<N>> |
| Encryption & key management | <<N>> |
| Network exposure | <<N>> |
| Identity & access | <<N>> |
| Tagging, naming, governance | <<N>> |
| Backup & resilience | <<N>> |
| Data services (DB, storage, queues) | <<N>> |
| Compute & containers | <<N>> |
| AI services | <<N>> |

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
| [cspm-rules-manager-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cspm-rules-manager-preview) | One CLI to collect, update and create Prisma policies, Cortex rules and AppSec rules |
| [cspm-rules-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cspm-rules-preview) | Consolidated, validated CSPM rule library for Cortex Cloud |
| [cortex-cloud-terraform-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cortex-cloud-terraform-preview) | Provisions the managed security service framework for a Cortex Cloud tenant, as code |
| [prisma-asset-reporter-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/prisma-asset-reporter-preview) | Per-asset compliance findings and vulnerabilities from Prisma Cloud, cross-cloud, by asset type or custom RQL |
| [cortexcloud-sentinel-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cortexcloud-sentinel-preview) | Microsoft Sentinel → AWS (OpenSearch + Grafana) data pipeline |

---

*Palo Alto Networks, Prisma Cloud, Cortex, Cortex Cloud, Cortex XDR and Cortex XSIAM are
trademarks of Palo Alto Networks, Inc. This is not an official Palo Alto Networks project.
Figures quoted describe specific engagements at the dates given.*
