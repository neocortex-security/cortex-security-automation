# FAQ

**Does it translate Prisma Cloud default policies, or only custom ones?**
Both run through the same engine. Default policies are usually the smaller problem:
Cortex Cloud ships equivalents for most of them, and the migration question there is
mapping, not translation. Custom policies are where the estate's intent lives and where
the tool earns its keep.

**What fraction translates without a human touching it?**
Depends on the corpus. See the README results table for measured figures. The honest
framing: on a new corpus expect the first pass to leave a noticeable minority in
*disagree*, converging over a small number of rounds as the anomaly classes are
worked. Each corpus makes the next one better because fixes land in the engine, not in
per-client patches.

**How is this different from the vendor's migration validation?**
Count-only validation accepts a rule when the two totals are close. This tool compares
populations (or per-account counts where no identifier exists), names the cause of
every disagreement, and tracks override count per release so you can see the engine
improving. It also reports source-side defects — RQL that cannot fire on Prisma —
rather than translating them faithfully into rules that cannot fire on Cortex either.

**Can I use it on an estate that is not migrating — just to author XQL from RQL?**
Yes; the validation half needs a Cortex tenant with the assets onboarded, but
translation alone does not.

**Which Prisma policy types are covered?**
`config from cloud.resource` is production. IAM, audit-event and network policies are
routed to correlation rules; see the scope table in the README for current status.

**Does it handle multi-resource RQL (joins)?**
Yes, including the fan-out and anti-join cases that are the most common source of
plausible-but-wrong translations. See EXAMPLES.md §3.

**What about alert exceptions / snoozed alerts?**
Carried over to Cortex issue exceptions through the public API, preserving reason and
expiry where present.

**What does it need to run?**
A machine with network access to both tenants, API credentials with read scope on
Prisma and read + rule-management scope on Cortex, and the tool's binary. No Python
install, no database, no cloud account.

**Does any data leave my environment?**
No. Queries go to the two vendor APIs and results stay on the operator's machine.

**Why is the source not public?**
Because the value is in the schema catalogue, the engine rule set and the override
library, all of which took many validation rounds against real tenants to build. The
methodology is public so the results can be judged; the implementation is licensed.

**Can my team get it?**
See [ACCESS.md](../../ACCESS.md).
