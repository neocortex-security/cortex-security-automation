# Usage

Operator-facing reference, public so a prospective user can see exactly what running
the tool involves. The binary's `--help` is authoritative for a given release.

## Credentials

One INI file **per customer**, holding both tenants, kept outside any repository:

```
~/.config/panw/<customer>.ini
```

```ini
[URL]
BaseURL = https://api-<tenant>.xdr.<region>.paloaltonetworks.com
# SSL_VERIFY = false        ; only for a tenant behind a private CA (or use --insecure)

[AUTHENTICATION]
ACCESS_KEY_ID = <id>
SECRET_KEY    = <cortex api key>

[VALIDATION]
XQL_TEST_RULE_ID = <uuid of a custom rule on asset_inventory, created for validation only>

[PRISMA]
BaseURL       = https://api<n>.prismacloud.io    ; the API endpoint, not the app.* console
ACCESS_KEY_ID = <prisma access key id>
SECRET_KEY    = <prisma secret key>
```

`--config <customer>.ini` on every run: a bare name is looked up in `~/.config/panw/`,
a path is used as given. Resolution order: `--config` → the path saved by `--setup` →
`API_config.ini` in the current folder. `--setup` (per subcommand) stores customer
name, output folder and default config so interactive runs need fewer flags.

`XQL_TEST_RULE_ID` has no default: a tenant without one fails validation rather than
silently passing it. Create the test rule once per tenant; never point it at a
production rule.

## Subcommands

```
cspm-rules-manager prisma      [--collect | --update | --create-standard | --setup]
cspm-rules-manager cortex      [--collect | --update | --create-standard | --setup]
cspm-rules-manager appsec      --cspm-excel … --appsec-excel … --output … [--validate | --resolve-cspm-ids]
cspm-rules-manager exceptions  [--collect | --update] [--dry-run]
cspm-rules-manager <subcommand> --help
cspm-rules-manager --version
```

Run a subcommand with no action flag and it presents a numbered menu (collect, update,
both, create standard, exit).

### `prisma` — Prisma Cloud policies

| Flag | Does |
|---|---|
| `--collect` | Export every policy of a custom compliance standard — RQL, mode, severity, labels, saved search, standard mapping — to a workbook |
| `--update` | From the reviewed workbook: create / update custom policies and saved searches, map policies to standards, apply labels |
| `--create-standard` | Create a custom compliance standard with requirements and sections from the template workbook |
| `--standard-id <uuid>` | Skip the interactive standard list |

### `cortex` — Cortex Cloud rules

| Flag | Does |
|---|---|
| `--collect` | Export CSPM and AppSec rules of a custom standard with their compliance mapping to a workbook |
| `--update` | From the reviewed workbook: create, patch, map and label rules — **each XQL is validated through the test rule before any production rule is touched** |
| `--create-standard` | Create (or extend) a custom compliance standard and its controls from the same template workbook as `prisma --create-standard`; idempotent, reuses controls by name |
| `--template`, `--standard-name`, `--standard-description`, `--labels` | Inputs for `--create-standard` |
| `--standard-id <uuid>` | Skip the interactive standard list |

Cortex has no requirement / section hierarchy — a standard is a flat set of controls,
each tagged with a category and subcategory — so each template row becomes one control
(`Requirement` → category, `Requirement_ID` → subcategory, control name
`<Requirement_ID> - <Section>`). The naming matches the Prisma side so a rule mapped on
both platforms carries the same control text.

### `appsec` — IaC rule pairing

| Flag | Does |
|---|---|
| `--cspm-excel`, `--appsec-excel`, `--output` | Pair translated CSPM rules with existing AppSec (IaC) rules — by name, then by asset type and keywords — or generate Checkov-style YAML from the XQL for rules with no match |
| `--validate` | Validate the generated YAML |
| `--resolve-cspm-ids` | Replace `cspmRuleId` placeholders with rule UUIDs once the CSPM rules are deployed |

### `exceptions` — alert exceptions → issue exceptions

| Flag | Does |
|---|---|
| `--collect` | Read Prisma alerts with status *dismissed* or *snoozed* (paged); one row per alert with alert, policy and resource detail; resolve each Prisma policy to its Cortex rule (rules workbook `PolicyID`, then name, then fuzzy name against all tenant rules); `Included = ADD` when found, `REVIEW` when not |
| `--update` | Create one Cortex issue exception per (rule, account, expiry, category), asset names chunked per exception; search-before-create on a deterministic external id so re-runs are idempotent; `ExceptionID` written back, `DONE` / `FAILED` per row |
| `--rules-workbook` | The `cortex --collect` or translator workbook used to resolve policy → rule |
| `--dismissed-expiry-days` | Expiry for dismissed alerts (snoozed alerts keep their snooze end) |
| `--match-on rule` | For asset-less issues, match on the rule only |
| `--strip-prefix` | Strip an organisation's naming prefix when matching renamed policies |
| `--chunk` | Asset names per exception |
| `--prisma-config` | When the Prisma credentials are in a separate file |
| `--customer` | Customer name used in output file names |

**Measured platform limit.** Cortex Cloud tenants accept about **10 exception creations
per rolling 24 hours** by default (measured on a lab tenant, Nov 2026). When the tool
hits it, it prints when the window reopens — computed from the creation timestamps
already in the workbook — and how many runs remain, so the next run is planned rather
than tried. For a migration of any size, have the tenant administrator raise the limit
before the day; unlimited, creation runs at roughly 10 s per exception.

## Common flags

```
--config <customer>.ini   credentials (required unless saved by --setup)
--dry-run                 print every call, change nothing
--yes                     skip the overwrite and collect-to-update confirmations
--standard-id <uuid>      skip the interactive standard list
--insecure                skip TLS verification (private CA)
--verbose, -v             API payload traces
```

## Workbook convention (shared by every subcommand)

- The **`Included`** column is the instruction: `CREATE_CUSTOM`, `UPDATE_CUSTOM`, `ADD`,
  `REMOVE`, `LABEL`. Rows marked `REVIEW`, `MAYBE` or `NO` are left alone.
- The **`Reason`** column is the tool's answer per row after a run.
- A timestamped backup is written before any workbook is rewritten.
- Outputs land in `data/`:
  `rules_<Customer>_<Standard>_latest.xlsx`, `exceptions_<Customer>_latest.xlsx`, …

## Rule test pipeline

```
testing/rule_test_pipeline.py --config <customer>.ini --rules <workbook>
testing/terraform/{aws,azure,gcp}
```

Provisions deliberately misconfigured resources in a **lab** account, waits for
collection, confirms each deployed rule fires on exactly that resource, destroys.
Stays a script (needs Terraform and the cloud CLIs beside it) and is not in the binary.

## Distribution

Single binary per platform (macOS arm64, Linux x86_64) built from the same source; no
Python on the operator's machine. Ships with an installation guide per platform and
the migration guides (rules; exceptions).
