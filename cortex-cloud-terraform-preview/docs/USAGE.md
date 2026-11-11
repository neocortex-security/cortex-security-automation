# Usage

## Layout of a tenant repository (yours, not this one)

```
tenants/
  prod/
    main.tf          # module "tenant" { source = …//modules/tenant?ref=vX.Y.Z }
    terraform.tfvars # the reviewed description of the tenant
    backend.tf       # remote state (S3 + lock, Azure blob, GCS)
  dev/
    …                # same module, different tfvars
shared/
  asset_groups.tf    # locals reused by both tenants
  rule_overrides.tf
  exceptions.tf
```

## Variables (module `tenant`)

<!-- Keep in sync with variables.tf; this is the public interface. -->

| Variable | Type | Purpose |
|---|---|---|
| `tenant_fqdn` | string | `api-<tenant>.xdr.<region>.paloaltonetworks.com` |
| `cloud_accounts` | map(object) | per-cloud onboarding: identifiers, scope (all / list), roles/permissions level, regions |
| `asset_groups` | map(object) | name, description, `type` (static \| dynamic), `xql` for dynamic |
| `standards` | list(string) | compliance standards to enable |
| `custom_standards` | map(object) | name, sections, requirements, mapped rule ids |
| `rule_overrides` | map(object) | by rule id: enabled, severity, scope |
| `custom_rules` | map(object) | name, xql, severity, asset types — typically from a cspm-rules workbook |
| `exceptions` | list(object) | rule id, scope, reason, expiry |
| `integrations` | object | per-integration settings; secrets via variables marked sensitive |
| `users`, `roles` | map(object) | principals, role bindings, scopes |
| `tags` | map(string) | applied where the API supports it |

Secrets (`auth_key`, integration tokens) come from the environment or a secrets
provider, never from `tfvars`.

## Workflow

```
terraform init
terraform plan  -var-file=terraform.tfvars -out=plan.bin   # review
terraform apply plan.bin
```

- **First apply** on an empty tenant: creates everything; expect <<N>> minutes for a
  3-cloud tenant, bounded by onboarding completion.
- **Subsequent plans**: show only drift. A plan with changes you did not make is a
  change someone made in the console.
- **Import** of an existing tenant: `scripts/import_tenant.sh` discovers current state
  and writes import blocks so the modules adopt what is there rather than recreating it.

## Cloud-side prerequisites

Onboarding needs resources in the cloud accounts themselves (roles, trust policies,
event/log sinks). The modules create those with the native providers (`aws`, `azurerm`,
`google`, `oci`) so one apply does both halves. Credentials for the cloud side are the
usual provider credentials.

## Versioning

Modules are tagged `vX.Y.Z`. Pin `ref=` in the source. Minor versions add coverage;
majors change the variables interface and ship with an upgrade note.
