# Catalogue

One line per rule. No XQL. Generated from the private library by
`scripts/make_catalogue.py` — do not edit by hand.

<!--
GENERATE, don't write. Columns: id · name · cloud · kind · asset types (count) ·
severity · frameworks · coverage · last validated. Sort by cloud then family.
Rule NAMES must be generic — a name like "<Client> landing-zone subnet policy" is a
client identifier. Rewrite before generation; the script should refuse names on the
deny-list.
-->

Last generated: <<YYYY-MM-DD>> · <<N>> rules

## AWS

| Id | Name | Kind | Asset types | Sev | Frameworks | Coverage | Validated |
|---|---|---|---|---|---|---|---|
| CR-AWS-LOG-0001 | <<…>> | posture | 1 | high | CIS 3.x | full | <<YYYY-MM>> |
| … | | | | | | | |

## Azure

| Id | Name | Kind | Asset types | Sev | Frameworks | Coverage | Validated |
|---|---|---|---|---|---|---|---|
| … | | | | | | | |

## GCP

…

## OCI

…

## Alibaba

…

## Kubernetes

…

## Partial-coverage rules

Rules whose control cannot be fully checked because a field is not collected. Listed
separately so the gap is visible.

| Id | Name | Cloud | What is checked | What is not |
|---|---|---|---|---|
| … | | | | |
