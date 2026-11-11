# FAQ

**Is this a replacement for the consoles?**
For bulk work, yes. For a single rule change the console is fine. The tool exists for
the cases where there are hundreds of objects, two platforms and more than one tenant.

**Can it break my tenant?**
Every write reads from a workbook whose `Included` column you set; rows marked
`REVIEW`, `MAYBE` or `NO` are never touched; there is no bulk-delete. On the Cortex side
each XQL is validated through a dedicated test rule before a production rule is
created or patched, and the tool refuses to run validation against a production rule.
`--dry-run` prints every call. Backups are written before any workbook is rewritten.

**What is the workbook convention?**
`Included` is the instruction (`CREATE_CUSTOM`, `UPDATE_CUSTOM`, `ADD`, `REMOVE`,
`LABEL`); `Reason` is the tool's answer per row. The same convention across all four
subcommands, so one review habit covers the whole migration.

**Does it handle Cortex compliance standards?**
Yes — `cortex --create-standard` builds a custom standard and its controls from the
same template workbook that `prisma --create-standard` uses, so a control carries the
same text on both platforms. Cortex standards are flat (controls with category and
subcategory, no requirement / section levels); the tool maps the template onto that
and uses the compliance API that rules can actually be mapped to — there are two, and
they do not see each other's standards.

**What does `appsec` actually do?**
Pairs each translated CSPM rule with an existing AppSec IaC rule by name, then by asset
type and keywords; generates Checkov-style YAML from the XQL for rules with no match;
validates the YAML; and resolves `cspmRuleId` placeholders to rule UUIDs once the CSPM
rules are deployed. It is a reviewed workflow, not an unattended one.

**Can it migrate exceptions / dismissed and snoozed alerts?**
Yes: `exceptions --collect` reads them from Prisma, resolves each policy to its Cortex
rule (workbook id, then name, then fuzzy name with an optional prefix strip for renamed
policies), and `exceptions --update` creates one issue exception per rule / account /
expiry group, idempotently. Snoozed alerts keep their snooze end as the expiry;
dismissed ones get a configurable expiry.

**Why does the exceptions migration talk about a quota?**
Because Cortex Cloud tenants accept roughly 10 exception creations per rolling 24
hours by default (measured on a lab tenant). The tool tells you when the window
reopens and how many runs remain, but for a migration of hundreds of groups the real
answer is to have the tenant administrator raise the limit before the day. With the
limit raised, creation runs at about 10 s per exception.

**What permissions does it need?**
Prisma: read for `--collect`, policy write for `--update` / `--create-standard`,
alerts read for `exceptions --collect`. Cortex: a standard or advanced API key with
rule read / write and compliance management; issue-exception management for
`exceptions --update`. Exact roles in the private repository's SECURITY.md.

**Does it need Python?**
No. It ships as a binary for macOS (arm64) and Linux (x86_64) with an installation
guide per platform. Python is only needed to run from source as a licensed user.

**Does it work with rql2xql output directly?**
Yes. The translator's deployable workbook is a `cortex --update` input without
reformatting, and `exceptions --collect` can use it as the rules workbook for
policy → rule resolution.

**Can my team get it?**
See [ACCESS.md](../ACCESS.md).
