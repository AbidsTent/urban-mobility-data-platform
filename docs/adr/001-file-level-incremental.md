# ADR-001: File-level incremental ingestion and month-based publication

- **Status:** Proposed — to be confirmed after P0 source investigation
- **Date:** 2026-11-05

## Context

TLC releases yellow-taxi trip data in monthly Parquet files. The source does not guarantee a stable trip identifier or a database-style change stream. Re-running a published file or receiving a revised file must not duplicate published monthly metrics.

## Options considered

1. Full history reload on each run — simple, but unnecessarily expensive and operationally weak.
2. Deduplicate or upsert rows by a constructed business-field hash — unsafe because distinct real trips can share the same visible values; a hash is not a proven trip key.
3. Track exact source file versions using SHA-256 and publish by month, replacing only an affected month after validation — preferred for monthly batch files.

## Proposed decision

Use `(source type, source month, SHA-256)` as a version identifier, keep raw versions, and use a published-partition registry. Identical published file versions are no-ops. A changed file is staged and validated before controlled replacement of the affected month. For MVP, accepted trips must satisfy the documented source pickup-month policy.

## Consequences

- Good: clear lineage, safer reruns and recovery, honest modeling of available input semantics.
- Trade-off: month reprocessing is coarser than true row-level CDC and requires careful multi-table publication strategy.

## Validation criteria

- Re-run an identical published file: no extra Gold rows or charges.
- Simulate changed file bytes: new version recognized and impacted month rebuilt.
- Simulate failure before publish: previous Gold snapshot remains usable.
- Record actual numbers and run IDs once implemented.
