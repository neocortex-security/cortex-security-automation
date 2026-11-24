# Changelog

Headline items only. Full release notes ship with the binaries.

## 1.2.1 — 2026-11
- The allowlist seed is now a CSV file beside the binary, not part of the tool: no
  addresses or identities are built in, and the file is validated in full before the
  workbook is touched (`--seed-file`)

## 1.2.0 — 2026-11
- `correlation` subcommand: correlation rules created, updated and exported from the
  same rules workbook as posture rules, including the translated IAM, audit-event and
  network policies; Real-time or Scheduled chosen per rule
- `allowlist` subcommand: known-good IPs and identities in two lookup datasets, managed
  from a workbook, with expiry dates, exact and pattern identities, and an optional seed
  of vendor scanner addresses
- Replaces the earlier one-off correlation-rule scripts; exclusions are no longer
  written into each query by hand
- Lookup behaviour in correlation rules measured on a live tenant and encoded

## 1.1.x — 2026-11
- `exceptions` subcommand: Prisma dismissed / snoozed alerts → Cortex issue exceptions;
  policy → rule resolution by id, name and fuzzy name; idempotent creation on a
  deterministic external id; tenant creation quota measured and planned around
- Whole-rule issue exclusion rules as an alternative mechanism (`--mechanism exclusion`)
- `cortex --create-standard` can create new control categories; system-default rules
  mapped with `ADD` / `REMOVE`; a standard's controls created in parallel
- Exceptions migration guide (Markdown + styled document)
- Organisation prefixes removed from all documentation examples

## 1.0.x — 2026-10
- Merge of the Prisma policies manager, the Cortex rules manager (v1.2) and the IaC
  manager into one binary with `prisma`, `cortex`, `appsec` subcommands
- `cortex --create-standard`: custom compliance standard and controls from the shared
  template; flat-model mapping; idempotent control reuse
- Credentials moved to one INI per customer in `~/.config/panw/`, outside any repo;
  `--setup` per subcommand
- Per-platform installation guides for the binary
- Migration guide covering the rules half of the Prisma → Cortex pipeline

## Cortex rules manager 1.2 — 2026-10 (pre-merge)
- Full code review: blocking error on one path fixed, YAML truncation fixed,
  HTTP-status-as-data bugs fixed, hard-coded tenant identifiers removed, TLS
  verification on by default with an explicit opt-out
- XQL validation through a dedicated test rule made mandatory
