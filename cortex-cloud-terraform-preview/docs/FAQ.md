# FAQ

**Is this an official Palo Alto Networks Terraform provider?**
No. There is no official provider for Cortex Cloud tenant configuration at the time of
writing. These modules use the public API directly. If an official provider appears, the
variables interface is designed so the modules can move onto it without changing your
`tfvars`.

**Can it adopt an existing tenant rather than build a new one?**
Yes — an import script discovers current configuration and generates import blocks.
Expect to spend the first review cycle deciding which console-made settings to keep.

**What happens if someone changes something in the console?**
The next `plan` shows it as drift. Whether you revert or adopt it is your call; the
modules do not auto-revert.

**Does it manage the cloud-side onboarding resources too?**
Yes, with the native providers, in the same apply. Onboarding is the part that most
often goes wrong by hand (missing permissions, wrong trust policy, regions left out), so
it is managed rather than documented.

**What about secrets?**
API keys and integration tokens are sensitive variables from the environment or a
secrets backend. They are not in `tfvars` and the modules do not write them to logs.

**How long does an apply take?**
Minutes for configuration; onboarding of a large cloud organisation takes as long as
Cortex takes to complete it, which the modules wait on with a timeout.

**Which Terraform / OpenTofu versions?**
Terraform ≥ <<1.x>>; OpenTofu ≥ <<1.x>>. Tested on both.

**Can I use just one sub-module?**
Yes; each is independent. `rules` alone is the common case for teams that onboarded by
hand but want rule enablement under version control.

**Can my team get it?**
See [ACCESS.md](../ACCESS.md). Delivery is as a private Git module source pinned by
tag.
