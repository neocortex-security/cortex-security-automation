# FAQ

**Where do the rules come from?**
Three sources, each marked in the entry's provenance: custom controls written by
organisations in Prisma Cloud and generalised (estate-specific values turned into
parameters); controls written to close gaps in the default Cortex Cloud rule set; and
framework requirements with no default rule. Nothing in the library is a copy of a
vendor default rule.

**Is anything in here a client's rule?**
The *intent* of many rules originated in a specific estate — that is the point of
consolidating. The *entries* are generalised: no account ids, regions, tag values,
naming conventions or other estate-specific literals; those are parameters. Rule names
are rewritten. Where a control was too specific to generalise, it is not in the
library.

**Is anything in here Palo Alto Networks' intellectual property?**
The library contains no Prisma Cloud default policy text and no Cortex Cloud default
rule text. XQL is the vendor's query language, as RQL was; writing rules in it does not
make the rules the vendor's. Where a library rule covers the same control as a default
rule, the entry says so and recommends the default.

**How often is it re-validated?**
Every library release re-runs every rule against the lab tenants; rules whose
population changes unexpectedly are investigated before release. Validation entries are
append-only so you can see a rule's history.

**What about controls Cortex Cloud cannot check?**
They are in the library as **partial**, with the gap written down. A migration that
silently drops the controls it cannot express is worse than one that documents them.
A separate coverage-gap document is available to licensed users: controls (including
AI-service controls) with no or partial Cortex default, and asset types with no posture
default.

**How do I deploy it?**
With `cspm-rules-manager cortex create` from the delivered workbook, after filling the
per-tenant parameter file. One reviewed run; refusals (rule API not yet accepting a
type id, for instance) are reported per rule.

**Can I get just the rules for one cloud?**
Yes; the workbook filters by cloud and family, and licensing is for the library as a
whole.

**Can my team get it?**
See [ACCESS.md](../ACCESS.md).
