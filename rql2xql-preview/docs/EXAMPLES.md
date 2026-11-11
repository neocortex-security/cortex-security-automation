# Worked examples

Each example shows the RQL, the XQL the tool produces, and the one thing about the
transformation that is not obvious. The examples are chosen for difficulty, not
convenience. Figures are from the reference tenants on the date shown; field paths are
illustrative of the pattern — the exact raw-field layout for a given asset type comes
from the schema catalogue in the full tooling.

<!--
HOW TO FILL THIS FILE
Take 6–10 entries from overrides.json / the engine test corpus that are ALREADY validated
(Agree, with both counts recorded). For each: paste the RQL (your own custom-policy RQL,
not a Prisma default policy's text), the final XQL, and the counts. Keep the "what is
non-obvious" line to one or two sentences. Delete this comment.
-->

---

## 1. List field read as a list, not a scalar

**RQL**
```
config from cloud.resource where api.name = 'aws-ec2-describe-security-groups'
  AND json.rule = ipPermissions[*].ipRanges[*] contains "0.0.0.0/0"
```

**XQL**
```
dataset = asset_inventory
| filter xdm.asset.provider = "AWS"
    and xdm.asset.type.name = "<<ASSET_TYPE>>"
| alter ranges = json_extract_array(xdm.asset.raw_fields, "$.<<path>>")
| filter array_any(ranges, to_string("@element") ~= "0\.0\.0\.0/0")
| fields xdm.asset.name, xdm.asset.realm
```

**What is non-obvious.** `[*]` in RQL quantifies over the list; a scalar
`json_extract_scalar` on the same path returns null and the rule silently matches
nothing. Nested quantifiers (`[*]…[*]`) cannot be expressed as nested `array_any` —
the inner list is serialised with `to_string` and matched with a regex.

**Measured** <<YYYY-MM-DD>>: Prisma <<N>> / Cortex <<N>>.

---

## 2. Absent field — the null arm

**RQL**
```
config from cloud.resource where api.name = '<<api>>'
  AND json.rule = <<field>> does not exist or <<field>> equals false
```

**XQL**
```
dataset = asset_inventory
| filter xdm.asset.type.name = "<<ASSET_TYPE>>"
| alter v = json_extract_scalar(xdm.asset.raw_fields, "$.<<field>>")
| filter v = null or lowercase(v) = "false"
| fields xdm.asset.name
```

**What is non-obvious.** RQL's `does not exist` and XQL's `= null` agree here, but the
reverse rule (`exists and equals true`) does **not** need a null arm, and adding one
inverts the test. Several engineering-supplied rewrites failed on exactly this. The
tool emits the null arm only for the negated form.

**Measured** <<YYYY-MM-DD>>: Prisma <<N>> / Cortex <<N>>.

---

## 3. Multi-resource join without fan-out

**RQL**
```
config from cloud.resource where api.name = '<<parent-api>>' as X;
config from cloud.resource where api.name = '<<child-api>>' as Y;
filter '$.X.<<key>> equals $.Y.<<key>> and $.Y.<<flag>> is false';
show X;
```

**XQL (shape)**
```
dataset = asset_inventory | filter xdm.asset.type.name = "<<PARENT>>"
| alter k = json_extract_scalar(xdm.asset.raw_fields, "$.<<key>>")
| join type = left (
    dataset = asset_inventory | filter xdm.asset.type.name = "<<CHILD>>"
    | alter k = json_extract_scalar(xdm.asset.raw_fields, "$.<<key>>"),
            flag = json_extract_scalar(xdm.asset.raw_fields, "$.<<flag>>")
    | filter lowercase(flag) = "false"
    | dedup k
  ) as y  k = y.k
| filter y.k != null
| fields xdm.asset.name
```

**What is non-obvious.** Without `dedup` on the child side the join returns one row per
child and the count is a multiple of the resource count. Where the RQL's `show X` means
"parents that have **no** qualifying child", the join becomes an anti-join
(`y.k = null`) — and the tool measures per account to confirm which reading Prisma
actually implements, because the two give the same tenant-level count on sparse data.

**Measured** <<YYYY-MM-DD>>: Prisma <<N>> / Cortex <<N>> (per-account: <<N>> of <<N>>
accounts agree).

---

## 4. Field casing that the API documentation gets wrong

**RQL**
```
config from cloud.resource where api.name = '<<api>>' AND json.rule = <<Field.SubField>> equals "x"
```

**XQL**
```
… json_extract_scalar(xdm.asset.raw_fields, "$.<<field.subField>>") …
```

**What is non-obvious.** The collected key casing differs from the vendor API reference
for a measurable fraction of asset types. A translation from documentation alone returns
null for the field and the rule matches nothing — which looks like "agree at zero" if
nobody checks that the type is populated. The schema catalogue in the tool is built from
collected data, not documentation; a casing audit corrected <<N>> paths across <<N>>
asset types.

---

## 5. Date arithmetic

**RQL**
```
… json.rule = <<dateField>> _DateTime.ageInDays > 90
```

**XQL**
```
| alter d = parse_timestamp("%Y-%m-%dT%H:%M:%E*S%Ez", json_extract_scalar(xdm.asset.raw_fields, "$.<<dateField>>"))
| filter timestamp_diff(current_time(), d, "DAY") > 90
```

**What is non-obvious.** Prisma's `ageInDays` on an **absent** field evaluates true, so
the RQL also flags resources that have no date at all. The tool measured that on the
reference tenants and reproduces it with an explicit null arm; the "obvious" translation
under-counts.

---

## 6. Prisma-refused: a rule that cannot fire

**RQL**
```
<<paste one of the corrected-RQL cases: tautology, uncollected attribute, wrong path>>
```

**Outcome.** The RQL returns every resource (or none) on Prisma regardless of
configuration. The tool reports it back with a corrected RQL and translates the
corrected intent. These are counted separately in the results table — they are a
finding about the source estate, not a translation result.

---

## 7. IAM policy → correlation rule

**RQL**
```
config from iam where dest.cloud.type = 'AWS' AND action.name CONTAINS ALL ( 'iam:*' ) …
```

**XQL (shape)**
```
dataset = <<identity-security dataset>>
| filter <<…>>
| comp count() by <<principal>>
```

**What is non-obvious.** There is no "advanced XQL" mode for this rule type in Cortex
Cloud, so the translation is deployed as a **correlation rule** rather than a posture
rule, with a different field set and schedule. The tool chooses the target from the
policy type.

---

## 8. Alert exceptions → issue exceptions

Not a translation — a carry-over. Prisma alerts with status `dismissed` / `snoozed` are
mapped to Cortex Cloud issue exceptions through the issue-exceptions API, preserving
reason text and expiry where Prisma has one. See the FAQ for the mapping grammar.
