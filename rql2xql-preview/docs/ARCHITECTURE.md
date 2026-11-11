# Architecture

```mermaid
flowchart LR
    subgraph Inputs
        P[Prisma Cloud tenant<br/>custom policies via API]
        F[Policy export<br/>CSV / XLSX / JSON]
    end

    subgraph Translator
        N[RQL normaliser]
        R[Schema-aware<br/>path resolver]
        E[Translation engine<br/>rule set]
        O[Override library]
        T[Policy-type router<br/>config / iam / event / network]
    end

    subgraph Validation
        Q1[Prisma query runner<br/>prod + dev]
        Q2[Cortex XQL runner<br/>prod + dev]
        C[Population / per-account<br/>comparison]
        A[Anomaly classifier]
    end

    subgraph Outputs
        W[Review workbook<br/>XLSX]
        D[Deployer<br/>posture rules / correlation rules]
        X[Exceptions carry-over<br/>alerts → issue exceptions]
    end

    P --> N
    F --> N
    N --> R --> E
    O -.-> E
    E --> T
    T --> Q2
    N --> Q1
    Q1 --> C
    Q2 --> C
    C --> A
    A --> W
    A -. engine fix / override .-> E
    C --> D
    P --> X
```

## Components

| Component | Role | Public? |
|---|---|---|
| RQL normaliser | Canonical form of the RQL text; the key for overrides and caching | no |
| Schema catalogue | Collected raw-field layout per asset type (not vendor documentation); drives path resolution and casing | no |
| Translation engine | Versioned rule set; each rule carries the measurement that justified it | no |
| Override library | Hand-written XQL keyed by normalised RQL, each with counts and date | no |
| Policy-type router | Chooses posture rule vs correlation rule and the source dataset | described here |
| Query runners | Async, rate-limited, retry on tenant concurrency limits; prod and dev in one pass | no |
| Comparison | Set difference on resource identifiers where available, else per-account counts | described in METHODOLOGY |
| Anomaly classifier | Assigns one of the named classes to each disagreement | classes public, rules private |
| Review workbook | Per-batch XLSX for client and engineering review | sample public |
| Deployer | Creates / updates rules through the Cortex public API | no |

## Runtime

- Python, distributed as a single binary per platform (macOS arm64, Linux x86_64); no
  Python install needed on the operator's machine.
- Credentials: per-tenant config files outside the repository; never on the command
  line.
- A full validation pass over ~500 policies against two Prisma and two Cortex tenants
  completes in <<N>> minutes, bounded by tenant query concurrency limits.

## Data handling

- Policy text and query results stay on the operator's machine; nothing is sent
  anywhere except the two vendor APIs.
- Review workbooks contain resource identifiers from the client's tenants and are
  treated as client data.
