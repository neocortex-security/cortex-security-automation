# Changelog

Headline items only. Full release notes are in the private repository.

<!--
Fill from the private repo's history. Keep to 3–6 bullets per release. Rule numbers,
register ids and ticket references stay out; describe the behaviour change.
-->

## 2.32.x — <<2026-10>>
- Engine rules for nested list regex, serialised-object matching, list-not-empty and
  object-casing cases; <<N>> overrides retired as a result
- Per-account comparison used to settle join vs anti-join readings
- Prisma-side RQL defects reported with corrected RQL

## 2.31.0 — <<2026-10>>
- Pull Prisma custom policies live from the tenant (`--prisma-policies`) instead of
  requiring an export file

## 2.29–2.30 — <<2026-09>>
- Translation of IAM, audit-event and network policy types routed to correlation rules
- Alert exceptions carried over to Cortex issue exceptions

## 2.2x — <<2026-08>>
- Dual-environment runs (prod + dev on both sides) in one pass
- Anomaly classification and threshold flag
- Retry/back-off on tenant concurrency limits

## 1.x — <<2026-06/07>>
- Schema catalogue built from collected data; casing audit across <<N>> asset types
- First client corpus validated end to end
