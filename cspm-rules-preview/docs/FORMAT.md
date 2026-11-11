# Library entry format

Each rule is one JSON object (and one workbook row). The schema is public so that a
prospective user can see what they would be deploying and how it is documented.

```json
{
  "id": "CR-AWS-LOG-0042",
  "name": "VPC flow logs not delivered to the central logging account",
  "intent": "Every VPC must have at least one flow log whose destination is the organisation's logging account.",
  "family": "logging-monitoring",
  "cloud": "AWS",
  "rule_kind": "posture",
  "asset_types": ["<<asset type id as the rule API accepts it>>"],
  "severity": "high",
  "xql": "dataset = asset_inventory | filter ... ",
  "parameters": {
    "logging_account_ids": {
      "type": "list[string]",
      "description": "Account ids that count as the central logging destination",
      "example": ["acct-1"]
    }
  },
  "reads": ["$.FlowLogs[*].LogDestination", "$.FlowLogs[*].DeliverLogsStatus"],
  "coverage": "full",
  "coverage_note": null,
  "frameworks": {"CIS-AWS": ["3.9"], "NIST-800-53": ["AU-12"]},
  "provenance": {
    "origin": "custom-control-generalised",
    "first_seen": "<<YYYY-MM>>",
    "generalised_from": "<<sector>> estate custom policy; account ids parameterised"
  },
  "validation": [
    {
      "date": "<<YYYY-MM-DD>>",
      "tenant_class": "lab | client-consented",
      "population": <<N>>,
      "compared_against": "Prisma RQL on same estate | manual sample of 20 | console rule",
      "result": "agree"
    }
  ],
  "status": "active",
  "version": "1.0.0",
  "changelog": [{"version": "1.0.0", "date": "<<YYYY-MM-DD>>", "note": "initial"}]
}
```

## Field notes

- **`parameters`** — anything estate-specific (account ids, approved regions, tag keys,
  key ARNs) is a parameter with a placeholder, never a literal. Deployment substitutes
  values from a per-tenant parameter file. This is what makes the library portable.
- **`reads`** — the raw-field paths the XQL depends on. When Cortex changes collection
  for an asset type, this is how affected rules are found.
- **`coverage`** — `full`, `partial` (XQL checks what is collectable; `coverage_note`
  says what is not), or `withdrawn`.
- **`validation`** — append-only. A rule with no validation entry in the last
  <<N>> months is flagged for re-validation in the release notes.
- **`rule_kind`** — `posture` (compliance rule over `asset_inventory`) or `correlation`
  (identity, audit-event or network datasets).

## Workbook

Same fields as columns; `xql` in a wrapped cell; `validation` flattened to the latest
entry plus a count. The workbook is what `cspm-rules-manager cortex create` consumes.
