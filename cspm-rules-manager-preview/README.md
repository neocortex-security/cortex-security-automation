# cspm-rules-manager — one CLI for Prisma Cloud policies, Cortex Cloud rules and AppSec rules

> **Preview repository.** Documentation, CLI reference, architecture and sample output.
> Binaries are in the private repository — see
> [Getting the full tooling](#getting-the-full-tooling).

![demo](docs/demo.gif)

## The problem

A CSPM migration or a multi-tenant Cortex Cloud estate means the same rule logic has to
be collected from, compared across, and pushed to several systems: Prisma Cloud custom
policies and compliance standards, Cortex Cloud compliance / posture rules and
correlation rules, and the IaC (Checkov-style) rules that enforce the same controls
before deployment. Each has its own API, its own object model and its own
rate-limiting. Doing it from the consoles does not scale past a few dozen rules;
doing it from ad-hoc scripts does not survive the second tenant.

## What cspm-rules-manager does

One binary, four subcommands, one workbook convention. Every change to a tenant goes
through a workbook a human has reviewed.

| Subcommand | Side | Does |
|---|---|---|
| `prisma` | Prisma Cloud | Export every policy of a custom standard with its RQL, mode, labels, saved search and standard mapping; create / update custom policies and saved searches; map to standards; create a custom standard from a template workbook |
| `cortex` | Cortex Cloud | Export CSPM and AppSec rules of a custom standard with their compliance mapping; create / patch / map / label rules — **every XQL validated through a dedicated test rule first**; create a custom standard and its controls from the same template |
| `appsec` | Cortex Cloud | Pair translated CSPM rules with existing AppSec IaC rules (by name, then asset type + keywords) or generate Checkov-style YAML from the XQL; validate; resolve rule-id placeholders after deployment |
| `exceptions` | both | Carry Prisma alert exceptions (dismissed / snoozed) to Cortex issue exceptions: collect, resolve policy → rule, review, create idempotently — with the tenant's creation quota measured and planned around |

A companion **rule test pipeline** (Terraform fixtures for AWS / Azure / GCP + runner)
provisions a deliberately misconfigured resource in a lab account, confirms the
deployed rule fires on exactly that resource, then destroys it.

```
cspm-rules-manager prisma      --collect | --update | --create-standard   --config <customer>.ini
cspm-rules-manager cortex      --collect | --update | --create-standard   --config <customer>.ini
cspm-rules-manager appsec      --cspm-excel … --appsec-excel … --output …  [--validate | --resolve-cspm-ids]
cspm-rules-manager exceptions  --collect | --update [--dry-run]           --config <customer>.ini
```

Run any subcommand without an action flag for a numbered menu.

## Results

<!-- Replace with dated figures; no client names. -->

| Task | Scale | Time |
|---|---|---|
| Collect a custom standard's Cortex rules with mapping, one tenant | <<N>> rules | <<N>> min |
| Create + map rules from a reviewed workbook, XQL validated first | <<N>> rules | <<N>> min, zero console edits |
| Create a custom compliance standard and its controls from the template | <<N>> controls | <<N>> min |
| Alert exceptions → issue exceptions, lab migration | <<20>> rule-account groups | <<N>> runs across the tenant quota; ~10 s per exception once lifted |
| Rule test pipeline, full run | <<N>> rules | <<N>> min |

## Where it fits

```
Prisma policies ──prisma --collect──► workbook ──rql2xql──► Cortex rules ──cortex --update──► Cortex tenant
       │                                  │                                        │
       │                                  └──appsec──► Checkov rules               └──test pipeline──► fires? ✓
       └──exceptions --collect──► review ──exceptions --update──► Cortex issue exceptions
```

`cspm-rules-manager` is the collection, deployment and testing half; translation is
[rql2xql](https://github.com/neocortex-security/cortex-security-automation/tree/main/rql2xql-preview). The two share config and
workbook formats.

## What is in this repository

| Path | Contents |
|---|---|
| [docs/USAGE.md](docs/USAGE.md) | CLI reference, credentials file layout, workbook convention, measured limits |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Subcommand → module → API map; the compliance-API and exceptions findings |
| [docs/SAMPLE_OUTPUT/](docs/SAMPLE_OUTPUT/) | Sanitised collect workbook, exceptions review workbook, dry-run output |
| [docs/FAQ.md](docs/FAQ.md) | Scope, limits, safety |
| [CHANGELOG.md](CHANGELOG.md) | Release history (headline items) |
| [ACCESS.md](ACCESS.md) | How to get the full tooling |

## What is **not** in this repository

The API client layer (including the request shapes the public documentation
does not spell out), the test-pipeline Terraform fixtures, the binaries, and the
per-platform installation and migration guides that ship with them.


---

## Getting the full tooling

This tooling was built in the course of Palo Alto Networks Professional Services
engagements and is made available to organisations working with me on a Cortex Cloud
deployment or migration. See [ACCESS.md](ACCESS.md).

## Related previews

| Repo | What it is |
|---|---|
| [rql2xql-preview](https://github.com/neocortex-security/cortex-security-automation/rql2xql-preview) | Prisma Cloud RQL → Cortex Cloud XQL translator with live validation |
| [cspm-rules-manager-preview](https://github.com/neocortex-security/cortex-security-automation/cspm-rules-manager-preview) | One CLI to collect, update and create Prisma policies, Cortex rules and AppSec rules |
| [cspm-rules-preview](https://github.com/neocortex-security/cortex-security-automation/cspm-rules-preview) | Consolidated, validated CSPM rule library for Cortex Cloud |
| [cortex-cloud-terraform-preview](https://github.com/neocortex-security/cortex-security-automation/cortex-cloud-terraform-preview) | Cortex Cloud tenant provisioning as code |
| [prisma-asset-reporter-preview](https://github.com/neocortex-security/cortex-security-automation/prisma-asset-reporter-preview) | Per-asset compliance findings and vulnerabilities from Prisma Cloud, cross-cloud, by asset type or custom RQL |
| [cortexcloud-sentinel-preview](https://github.com/neocortex-security/cortex-security-automation/cortexcloud-sentinel-preview) | Microsoft Sentinel → AWS (OpenSearch + Grafana) data pipeline |

---

*Palo Alto Networks, Prisma Cloud, Cortex, Cortex Cloud, Cortex XDR and Cortex XSIAM are
trademarks of Palo Alto Networks, Inc. This is not an official Palo Alto Networks project.
Figures quoted describe specific engagements at the dates given.*
