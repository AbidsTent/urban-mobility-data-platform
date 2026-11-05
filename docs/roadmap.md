# Roadmap and phase gates

The full requirements, tests, architecture and acceptance criteria are in [PROJECT_SPEC.md](PROJECT_SPEC.md). Only mark phases complete after their proof is recorded.

| Phase | Primary work | Exit evidence | Status |
|---|---|---|---|
| P0 | Repo foundation, source documentation, one-month profiling, initial ADR, environment verification | Actual January 2025 schema/profile; documented rule decisions; reproducible environment smoke test | In progress |
| P1 | Robust Python downloader, SHA-256 manifest, file selection and state transitions | Identical input replay test and failure handling | Planned |
| P2 | Versioned raw Bronze in PySpark + Delta Lake | Raw-to-Bronze count and lineage reconciliation | Planned |
| P3 | Silver standardization and quarantine | Valid/invalid fixture tests and real quality report | Planned |
| P4 | Changed-file publication and rollback | New/changed/identical month tests and controlled failure demo | Planned |
| P5 | Gold facts, zones, dates and business aggregates | No-fanout and financial/count reconciliation tests | Planned |
| P6 | Orchestration, retry, structured logs, GitHub Actions | Actual CI passing, recovery runbook and fault-injection demonstration | Planned |
| P7 | Gold-based Power BI reporting | Dashboard metrics reconciled with SQL | Planned |
| P8 | Optional weather/cloud/benchmarks and portfolio polish | Documented optional demo and measured benchmark | Planned |

## P0 issue shortlist

1. `P0-01` Publish public repo, file structure, documentation and issue/PR conventions.
2. `P0-02` Read official TLC sources and download January 2025 locally (do not commit data).
3. `P0-03` Inspect exact source schema and profile actual quality values; record quantitative evidence.
4. `P0-04` Propose at least five data-quality decisions with rationale and source links.
5. `P0-05` Decide file-level incremental strategy (ADR-001) and validate local Spark/Delta compatibility.

Do P0-01 before P0-02. Check off P0-01 only **after the repository is actually published**.
