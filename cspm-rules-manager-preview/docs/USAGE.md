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
cspm-rules-manager correlation [--collect | --update] [--dry-run]
cspm-rules-manager allowlist   [--collect [--seed-file <csv>] | --update] [--dry-run]
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
| `--mechanism exclusion` | Create one whole-rule issue exclusion per Cortex rule instead of per-resource exceptions. It silences every issue of the rule, in all accounts, with no expiry, so it is wider than the Prisma dismissals it came from; `--exclusion-status disabled` creates them disabled for review |

**Measured platform limit.** Cortex Cloud tenants accept about **10 exception creations
per rolling 24 hours** by default (measured on a lab tenant, Nov 2026). When the tool
hits it, it prints when the window reopens — computed from the creation timestamps
already in the workbook — and how many runs remain, so the next run is planned rather
than tried. For a migration of any size, have the tenant administrator raise the limit
before the day; unlimited, creation runs at roughly 10 s per exception.

### `correlation` — correlation rules from the rules workbook

The workbook is the same rules layout `cortex --update` reads. A row is a correlation
rule when its detection method marks it as an IAM, audit-event or network rule (written
by the rql2xql translator) or when it came from `correlation --collect`. `cortex
--update` skips those rows; `correlation --update` takes only those rows.

| Flag | Does |
|---|---|
| `--collect` | Export the tenant's correlation rules to a workbook, with the allowlist stages taken back out of the query so the sheet always holds the clean XQL |
| `--update` | Create (`CREATE_CUSTOM`) or update (`UPDATE_CUSTOM`) the marked rows; rule id and execution mode are written back |
| `--workbook` | The translated workbook or a `--collect` export |
| `--allowlist-default none\|auto` | What a blank `Allowlist` cell means |
| `--no-allowlist`, `--no-patterns` | Deploy with no allowlist stage; or with exact identities only |
| `--ip-field`, `--identity-field` | Field names for datasets without built-in defaults |
| `--scheduled-only` | For tenants without Real-time correlation |
| `--disabled` | Create rules disabled, to enable after review |
| `--retry-failed`, `--dry-run` | Re-run failed rows; write the payloads to JSON and change nothing |

Optional per-row columns: severity, enabled, MITRE tactics, execution mode, search
window, schedule, suppression fields and duration, alert domain and category, and
`Allowlist` (`none`, `ips`, `identities`, `both`, `auto`).

**Real-time or Scheduled.** A rule is Real-time when its query allows it. A rule that
reads the allowlist cannot be Real-time, so it is deployed Scheduled, hourly, with a
search window equal to the schedule; detection is then up to an hour behind ingestion.
Creating a rule whose name is already on the tenant fails, so there are no duplicates.

### `allowlist` — known-good sources that correlation rules ignore

Two lookup datasets, one for IP addresses and one for identities, managed from a
workbook with one sheet each.

| Flag | Does |
|---|---|
| `--collect` | Export both datasets to the workbook |
| `--collect --seed-file <csv>` | Also append the entries of a seed file as `ADD` rows. A seed file of vendor scanner and platform egress addresses by region, and vendor role-name patterns, ships beside the binary. `--seed-sources` and `--seed-regions` narrow it |
| `--update` | `ADD` writes or updates an entry, `REMOVE` deletes it; the tool answers `DONE` / `REMOVED` / `FAILED` with the reason |
| `--ips-dataset`, `--identities-dataset` | Rename the datasets (pass the same names to `correlation`) |

- **Expiry.** Each entry can carry an expiry date. The rules read it, so an expired
  entry stops excluding without anyone removing it.
- **Identity kind.** `exact`, or `pattern` (a regular expression). A pattern that
  matches everything is refused.
- **An empty allowlist excludes nothing.** The rules keep running and alert on
  everything again; `--update` warns when a dataset ends up empty.
- **Anyone who can edit the two datasets can silence the rules that read them.** Treat
  them as detection content: restrict who can edit, and fill in comment and expiry.
- **Nothing is built in.** The tool carries no addresses or identities of its own;
  the seed is a plain CSV you can read and edit (type, value, kind, source, region,
  comment, expiry). The whole file is checked before the workbook is touched: a CIDR,
  a non-address, an invalid or match-everything pattern, or an unknown type stops the
  run.
- Published vendor egress addresses change; check the seed file against the vendor
  documentation and edit it there. Customer identities do not belong in it; add them
  as workbook rows.

**Three mechanisms, three names.** An issue *exception* suppresses posture issues per
resource with an expiry (`exceptions`). An issue *exclusion* rule suppresses a whole
rule's issues (`exceptions --mechanism exclusion`). The *allowlist* is what correlation
rules read so they never alert on known-good sources (`allowlist`, `correlation`).

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
  `REMOVE`, `LABEL`. Rows marked `REVIEW`, `MAYBE` or `NO` are left alone. The same
  convention drives the correlation and allowlist workbooks.
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
