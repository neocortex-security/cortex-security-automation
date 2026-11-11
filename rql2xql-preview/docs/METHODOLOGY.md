# Methodology — what "validated" means here

The purpose of this document is to let a reader judge the results table in the README
without access to the tooling. Nothing here requires the private repository to
understand; everything here requires it to reproduce.

## 1. The loop

```
 Prisma policies ──► translate ──► XQL candidate
                                       │
              ┌────────────────────────┴─────────────────────────┐
              ▼                                                  ▼
     run RQL on Prisma tenant                          run XQL on Cortex tenant
     (prod + dev)                                      (prod + dev)
              │                                                  │
              └──────────────► compare populations ◄─────────────┘
                                       │
                         agree ────────┼──────── disagree
                           │                        │
                     deploy / record         classify anomaly
                                                    │
                                       ┌────────────┴────────────┐
                                       ▼                         ▼
                               engine fix (rule)          override entry
                                       │                         │
                                       └──────── re-run ◄────────┘
```

One pass over a corpus is a **round**. A rule leaves the loop when it reaches one of
the terminal states below.

## 2. Terminal states

| State | Meaning |
|---|---|
| **Agree** | XQL and RQL return the same population (same resource identifiers), or the same count where the dataset exposes no stable identifier |
| **Agree at zero** | Both sides return nothing; checked that the asset type is populated on both tenants so the zero is real |
| **Platform gap** | The RQL reads a field Cortex does not collect for that asset type. Documented per rule; the XQL returns the nearest measurable check and the gap is stated in the rule description |
| **Prisma refused** | The RQL itself cannot run or cannot fire on Prisma (invalid path, tautology, never-populated attribute). Reported back with a corrected RQL; the XQL is written to the corrected intent |
| **Disagree** | Open. Counted against the tool until resolved |

"Within threshold" in the README means **Agree + Agree at zero + Platform gap** as a
share of all rules in the corpus, with Prisma-refused rules excluded from the
denominator and reported separately.

## 3. Why populations, not counts

A count match is necessary, not sufficient. Observed on real corpora:

- a rule that matched to within 1 % on count while flagging a **different set** of
  resources, because a list field was read as a scalar on one side;
- a rule where the deployed XQL flagged every resource of a type (count 1 140) and the
  correct form flagged 24 — a count check against a stale Prisma figure passed it;
- joins that fan out (one row per association instead of per resource) and inflate the
  count until deduplicated.

Where the dataset exposes a resource identifier on both sides, the comparison is a set
difference. Where it does not, counts are compared **per account / subscription /
project**, which exposes most fan-out and scoping errors that a tenant-level total hides.

## 4. Anomaly classes

Every disagreement is assigned one class before anyone proposes a fix. The classes are
stable across corpora, which is what makes the engine improve rather than accumulate
one-off patches.

| Class | Typical cause | Typical resolution |
|---|---|---|
| Collection gap | Cortex does not collect the field / sub-resource / region | Platform gap; nearest check |
| Path casing | Raw-field key casing differs from the API documentation | Schema catalogue correction (engine) |
| List vs scalar | RQL `[*]` / `contains` semantics vs XQL array functions | Engine rule |
| Join fan-out | Multi-resource RQL translated to a join that multiplies rows | Dedup stage (engine) |
| Null arm | Absent field: RQL treats as true/false differently from XQL | Explicit null arm (engine) |
| Scoping | Account-group or region scoping present on one side only | Per-account comparison; scope clause |
| Serialised nested | Nested object serialised as a string in the dataset | Regex over serialised form (override) |
| Prisma-side defect | The RQL never fires, or reads an uncollected path on Prisma too | Report back; corrected RQL |
| Engine refusal | Cortex rule API rejects the type id or construct | Wait / alternative construct |

Counts per class per round are in the sample anomaly report under `SAMPLE_OUTPUT/`.

## 5. Overrides vs engine fixes

An **override** is a hand-written XQL keyed to the normalised RQL text. It is the
fastest way to close a rule and the slowest way to improve the tool. The working rule:

- an override is accepted only with a measurement attached (both counts, the date, the
  tenant class);
- when three or more overrides share a cause, the cause becomes an engine rule and the
  overrides are removed;
- overrides are re-validated on every engine release; an override that the engine now
  gets right on its own is deleted.

Override count is therefore reported per release and expected to **fall**.

## 6. What is measured each release

- Rules per terminal state, per corpus
- Overrides in force (and removed this release)
- Rules changed by the release, with old and new counts on the reference tenants
- Prisma-refused rules reported back, with the corrected RQL

## 7. Limits

- The comparison is only as good as the tenants: a dev tenant with partial onboarding
  produces "agree at zero" that is not evidence. The tool checks that the asset type is
  populated before accepting a zero, but it cannot detect a partially-onboarded account.
- Prisma Cloud and Cortex Cloud do not collect on the same schedule. Short-lived
  resources produce transient disagreements; the tool re-runs before classifying.
- The Cortex rule API's accepted asset-type identifiers lag the dataset in places; a
  validated XQL can still be refused at deployment. These are tracked as engine
  refusals, not translation failures.
