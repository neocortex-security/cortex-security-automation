# cortex-cloud-terraform — Cortex Cloud tenant provisioning as code

> **Preview repository.** Module interface, coverage, architecture and a sample plan.
> Module source is in the private repository — see
> [Getting the full tooling](#getting-the-full-tooling).

![demo](docs/demo.gif)

## The problem

Standing up a Cortex Cloud tenant properly — cloud accounts onboarded with the right
scopes, asset groups, compliance standards, rule enablement, exceptions, integrations,
users and roles, notification routing — is a few hundred console actions. Done by hand
it is slow, undocumented and different on every tenant. Done again for a dev tenant,
it drifts from prod within a week.

## What cortex-cloud-terraform does

Terraform modules that take a tenant from empty to configured, idempotently, from a
reviewed `tfvars` file:

```hcl
module "tenant" {
  source = "git::ssh://git@github.com/neocortex-security/cortex-cloud-terraform.git//modules/tenant?ref=v<<X.Y.Z>>"

  tenant_fqdn   = var.tenant_fqdn
  cloud_accounts = {
    aws   = { org_id = "o-xxxx", roles = ["read", "remediation"], regions = "all" }
    azure = { tenant_id = "…", subscriptions = "all" }
    gcp   = { org_id = "…" }
  }
  asset_groups   = local.asset_groups
  standards      = ["CIS-AWS-3.0", "ISO-27001-2022", "<<custom>>"]
  rule_overrides = local.rule_overrides
  exceptions     = local.exceptions
  integrations   = { jira = {…}, slack = {…}, siem = {…} }
}
```

`terraform plan` shows exactly what will change on the tenant; `apply` makes it so;
re-running on a drifted tenant shows the drift.

## Coverage

<!-- Mark each row: ✅ managed · 🟡 partial (note) · ⬜ not yet · — not exposed by the API -->

| Area | Resources | Status | Note |
|---|---|---|---|
| Cloud account onboarding | AWS org/account, Azure tenant/subscription, GCP org/project, OCI tenancy | <<✅>> | includes the cloud-side IAM/role stack via native providers |
| Asset groups | static and dynamic (XQL-defined) groups | <<✅>> | |
| Compliance standards | enable/disable, custom standards with sections and requirements | <<✅>> | |
| Rules | enablement, severity overrides, custom rules (XQL) | <<✅>> | custom rule bodies from the cspm-rules library |
| Issue exceptions | scoped exceptions with expiry | <<✅>> | |
| Users, roles, RBAC scopes | | <<🟡>> | <<note>> |
| Integrations | ticketing, chat, SIEM forwarding, email | <<🟡>> | <<note>> |
| Notification / alert routing | | <<⬜>> | |
| Data collectors / log sources | | <<⬜>> | |
| Dashboards and reports | | <<—>> | import via API outside Terraform |

## Results

<!-- Dated, no client names. -->

| Scenario | Scale | Console (estimated) | Terraform |
|---|---|---|---|
| New tenant, 3 clouds, <<N>> accounts, <<N>> standards, <<N>> custom rules | | <<N>> days | <<N>> min apply |
| Dev tenant mirrored from prod | | not practical | same `tfvars`, <<N>> min |
| Drift check, weekly | | manual audit | `plan`, <<N>> s |

## How the modules talk to Cortex

Cortex Cloud has no official Terraform provider for tenant configuration. The modules
use <<a thin custom provider | the HTTP/REST provider pattern | external data sources>>
over the Cortex public API with idempotency and drift detection handled per resource
type. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) — this is the part that took the
time and the part that is private.

## What is in this repository

| Path | Contents |
|---|---|
| [docs/USAGE.md](docs/USAGE.md) | Variables interface, tfvars shape, workflow |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Module layout and the API layer |
| [docs/SAMPLE_OUTPUT/](docs/SAMPLE_OUTPUT/) | A sanitised `plan` and a drift report |
| [docs/FAQ.md](docs/FAQ.md) | State, secrets, drift, API limits |
| [ACCESS.md](../ACCESS.md) | How to get the modules |


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
| [cortex-cloud-terraform-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cortex-cloud-terraform-preview) | Cortex Cloud tenant provisioning as code |
| [prisma-asset-reporter-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/prisma-asset-reporter-preview) | Per-asset compliance findings and vulnerabilities from Prisma Cloud, cross-cloud, by asset type or custom RQL |
| [cortexcloud-sentinel-preview](https://github.com/neocortex-security/cortex-security-automation/tree/main/cortexcloud-sentinel-preview) | Microsoft Sentinel → AWS (OpenSearch + Grafana) data pipeline |

---

*Palo Alto Networks, Prisma Cloud, Cortex, Cortex Cloud, Cortex XDR and Cortex XSIAM are
trademarks of Palo Alto Networks, Inc. This is not an official Palo Alto Networks project.
Figures quoted describe specific engagements at the dates given.*
