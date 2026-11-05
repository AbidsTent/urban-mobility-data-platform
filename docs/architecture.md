# Proposed architecture — P0

**Status:** design proposal, not a description of an implemented system.

```mermaid
flowchart LR
    SRC[TLC monthly yellow-taxi Parquet] --> ING[Python download, verify, file manifest]
    ZONE[Taxi zone CSV] --> ING
    ING --> BR[Bronze: original file versions, raw schema]
    BR --> CONTRACT[Contract gate]
    CONTRACT --> SILVER[Silver: accepted, standardized trips]
    CONTRACT --> QUAR[Quarantine + DQ summary]
    SILVER --> GOLD[Gold: dim_zone, dim_date, fact_yellow_trip]
    GOLD --> AGG[Gold data marts: daily zone and monthly KPI]
    AGG --> BI[Power BI]
    OPS[Orchestration, logs, tests and recovery] -.-> ING
    OPS -.-> SILVER
    OPS -.-> GOLD
```

## Architectural boundaries

- **Unit of ingestion:** an immutable version of a published TLC source-month file, identified by source type, month and SHA-256 hash.
- **Unit of reporting publication:** source month, with validated replacements and a published-version pointer/manifest.
- **Bronze:** preserves raw input versions and their lineage.
- **Silver:** applies contracts, cleaning, accepted vs. quarantined rules, and reconciliation.
- **Gold:** contains dimensional model **and** business-ready aggregates, not a separate mandatory post-Gold modeling stage.
- **No row-level CDC claims:** TLC monthly Parquet files do not supply a guaranteed trip ID or source change stream.
- **Optional future weather data:** joins at daily grain with explicit join-cardinality checks.

## Design decisions to verify

See [ADR 001](adr/001-file-level-incremental.md). Before implementation, confirm local Spark/Delta/Java compatibility, available disk/RAM, source schema, data-quality policy, and publish atomicity guarantees.

## Planned operations

Manifest tracking; structured run logs; contract gates; quarantine; per-month reconciliation; safe replays; failure recovery; CI. These are future deliverables, **not** deployed functionality at P0.
