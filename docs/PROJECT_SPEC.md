# Urban Mobility Intelligence Platform
## Master Project Specification, Learning Plan, and Acceptance Criteria

**Version:** 1.0  
**Prepared:** November 2, 2026  
**Status:** Proposed — not yet implemented  
**Project type:** End-to-end, production-oriented batch data engineering portfolio project  
**Initial domain:** New York City yellow-taxi trips  
**Core technology:** Python, PySpark, Delta Lake, SQL, orchestration, automated tests, GitHub Actions  
**Business output:** Trusted transportation data marts and a focused Power BI dashboard

> **Working rule:** This is a build specification, not a claim that the system exists. Check off requirements only after implementation and evidence. Source schemas, quality issues, record counts, and performance results must be inspected and measured, never assumed.

---

# 1. Executive summary

Build a repeatable data platform that processes real New York City Taxi & Limousine Commission (TLC) yellow-taxi trip records, produces reliable and documented analytics, and recovers correctly when data or processing fails. Extend the platform with taxi-zone reference data and historical weather data after the core pipeline is dependable.

The important difference from a classroom-style Medallion project is **operational depth**: source discovery, file tracking, batch idempotency, data contracts, PySpark transformations, quarantine rules, incremental monthly ingestion, backfills, partition replacement, orchestration, validation, run logs, automated testing, CI, and a documented incident-recovery demo.

**North-star deliverable:** A reviewer can clone the repository, run a documented sample workflow, inspect tests and data-quality reports, explain the Gold model, and watch a failed batch recover without duplicate reporting records.

## 1.1 Why this project exists

Existing SQL Server data-warehouse experience already demonstrates Bronze/Silver/Gold, cleaning, T-SQL, dimensional modeling, analytical SQL, and reporting. The new project must provide **new evidence of engineering ability** rather than repeating another full-refresh warehouse. Specifically, it should demonstrate:

1. Python-based source ingestion instead of only manual bulk imports.
2. PySpark/Delta proficiency on authentic multi-million-row Parquet data.
3. File-level incremental processing and safe replay.
4. Explicit data-quality policy, quality reporting, and rejected-row preservation.
5. Scheduling, logging, testing, CI, and incident recovery.
6. Measured performance and explainable architectural trade-offs.

## 1.2 Intended job alignment

Primary: Junior Data Engineer, Data Integration Analyst, ETL Developer, Analytics Engineer.  
Secondary: BI Developer, Reporting/Data Analyst roles requiring ETL and data modeling.

**Not a guarantee:** Completing a project does not guarantee hiring. The deliverables are meant to make engineering skills visible, verifiable, and easier to discuss in technical interviews.

---

# 2. Business case and stakeholders

## 2.1 Scenario

A fictional transportation analytics organization receives monthly, imperfect trip files from an external regulator. Operations and finance teams want consistent monthly trip metrics, reliable zone-based comparisons, and insight into whether trip demand varies with weather. Files may arrive late, be republished, contain unexpected field changes, and include logically suspicious records.

The organization needs an engineering platform that updates published reports **without manual copying**, silently lost records, or accidental double-counting.

## 2.2 Stakeholders and decisions

| Stakeholder | Questions the platform must support | Output |
|---|---|---|
| Operations analyst | Which pickup zones and hours have the highest demand? | Trips by zone, date, hour |
| Finance analyst | How do reported charges vary by month and zone? | Monthly charge totals and averages |
| Data quality owner | What fraction of each input file was accepted, quarantined, or excluded? | Quality and reconciliation report |
| Data platform owner | Did a batch succeed? Can I retry or backfill it safely? | Run logs, manifest, recovery runbook |
| Analytics manager | Does rainy weather correlate with different trip demand? | Optional weather-enriched analysis |

## 2.3 Business questions

- How many accepted yellow-taxi trips occurred per day and month?
- Which pickup boroughs/zones contributed the most recorded trips?
- How do average trip distance, duration, fare amount, and total passenger charges vary by time and zone?
- What percentage of records violated quality rules, and why?
- Did adding another month alter previously published months unexpectedly?
- How do trip volumes differ between rainy and dry dates, while recognizing that correlation is not causation?

**Metric wording:** `total_amount` means the amount charged to passengers as reported by TLC, **not** operator net revenue or profit. Cash tips are not included in the source's `tip_amount` or `total_amount` definitions. Do not mislabel the sums as net revenue.

---

# 3. Goals, success metrics, and boundaries

## 3.1 Project goals

**G1 — Correctness:** Build source-to-Gold lineage with reproducible row-count reconciliation.  
**G2 — Repeatability:** Ingest multiple monthly files through a command/configuration, not manual copying.  
**G3 — Safe updates:** Detect already-processed files and process changed file versions without double-counting.  
**G4 — Operational readiness:** Provide orchestrated tasks, useful logs, retries, recovery guidance, and tested failure cases.  
**G5 — Analytics:** Deliver a documented, trustworthy Gold star schema and useful Power BI report.  
**G6 — Portfolio evidence:** Publish code, tests, design decisions, diagrams, benchmark results, and a short walkthrough.

## 3.2 Definition of a successful MVP

The MVP is considered complete when **all** are demonstrated:

- [ ] Three distinct yellow-taxi source months are ingested from official TLC Parquet files.
- [ ] A source manifest records the URI, source month, hash, size, ingestion time, run ID, and processing state.
- [ ] Identical file replay produces **zero additional published trip records**.
- [ ] Changed bytes for a previously processed month are identified as a new source version.
- [ ] Critical invalid rows are quarantined with reason codes; raw source evidence remains available.
- [ ] Counts reconcile for each processed file: raw = accepted + quarantined + explicitly excluded, under documented classification rules.
- [ ] A Gold fact table and zone/date dimensions support reconciled monthly KPIs.
- [ ] A deliberate processing failure does not mark incomplete data as published; rerun succeeds.
- [ ] Automated unit and pipeline tests run in CI on small fixtures.
- [ ] An independent reader can follow README setup, view a run log, and execute the demonstration.

## 3.3 Stretch outcomes (not needed to ship MVP)

- Historical weather enrichment and a weather comparison page.
- 6–12 historical months, measured at larger scale.
- A Databricks-hosted execution of part or all of the pipeline.
- Schema-change compatibility tests across multiple source years.
- An intentionally simulated change-event source for practicing true row-level CDC and SCD Type 2 (clearly labeled as a simulation).
- Optional green-taxi ingestion as a separately modeled source after contract comparison.

## 3.4 Deliberate exclusions

No Kafka, Kubernetes, real-time streaming, vehicle-level tracking, passenger identities, ML predictions, automated cloud infrastructure provisioning, or unsupported claims of five-nines availability. No attempt to infer driver profit, actual demand unmet by taxis, trip uniqueness without source IDs, or causal weather effects.

---

# 4. Authoritative data sources

| ID | Source | What it provides | Cadence and caveats | Scope |
|---|---|---|---|---|
| SRC-01 | NYC TLC Trip Record Data | Yellow-taxi monthly trip Parquet | Posted monthly, usually after ~2 months; completeness/accuracy not guaranteed; schemas may vary | Required |
| SRC-02 | TLC Taxi Zone Lookup CSV | Location ID, borough, zone, service zone | Reference data; validate uniqueness before joining | Required |
| SRC-03 | Open-Meteo Historical Weather API | Historical temperature and precipitation for selected NYC coordinates | Modeled/reanalysis weather, not observed taxi-level conditions; respect terms/attribution | Optional Phase 2 |

**Official TLC source:** https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page  
**Yellow trip field dictionary:** https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf  
**Official zone CSV:** https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv  
**Weather docs:** https://open-meteo.com/en/docs/historical-weather-api

### Recommended initial source window

Begin with **January–March 2025 yellow taxi**. This is a manageable, consistent starting period and includes the 2025 `cbd_congestion_fee` field. First run only January 2025, then add February and March. Do not download all historical years before the MVP is proven.

Source link example (verify availability when executing):  
`https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2025-01.parquet`

**Source-use conditions:** Keep source attribution in the README. Do not commit large raw datasets to Git. Verify applicable TLC reuse terms. For Open-Meteo, follow its non-commercial API usage terms and CC BY 4.0 attribution when showing weather-derived information.

---

# 5. Requirements catalog

Each ID should map to an issue, a test, and evidence in the finished repository.

## 5.1 Functional requirements

| ID | Requirement | Verification / acceptance |
|---|---|---|
| FR-01 | Configure which months and source type to process | CLI/config accepts an explicit month list; an unknown month fails clearly |
| FR-02 | Download/retrieve original files reliably | Detect HTTP/download failure; record URI, size, SHA-256, retrieval time |
| FR-03 | Maintain a source-file manifest and batch state | New, processing, accepted/published, failed, skipped states visible |
| FR-04 | Preserve raw inputs and source metadata | Bronze retains original values and source lineage, without destructive cleaning |
| FR-05 | Validate file and column contracts | Missing critical columns block publication; unexpected columns recorded |
| FR-06 | Perform silver standardization in PySpark | Types, normalized names, derived duration, validation flags handled consistently |
| FR-07 | Quarantine critical invalid records | Rejected rows retained with one or more documented reason codes |
| FR-08 | Record quality counts by source version | Raw, accepted, quarantined, excluded counts and rates available |
| FR-09 | Support incremental **file-level** ingestion | Unchanged file skipped; new months processed without full-history rebuild |
| FR-10 | Support safe replacement of a changed month | Recompute impacted month's published outputs using versioned input; avoid partial publish |
| FR-11 | Build conformed analytical Gold tables | Fact grain and dimension keys documented; expected joins do not multiply rows |
| FR-12 | Produce reconciled KPI aggregates | Gold monthly counts and sums reconcile with valid Silver population |
| FR-13 | Orchestrate dependencies and retries | Defined DAG/task order; failure visible; rerun from safe state |
| FR-14 | Provide machine-readable pipeline logs | Run ID, phase, status, start/end, counts, duration, reason on failure |
| FR-15 | Automate critical tests | A CI workflow runs relevant unit, contract, and fixture tests |
| FR-16 | Support reproducible backfill | Explicit month range can be rebuilt using the intended source versions |
| FR-17 | Export business-ready analytics | Power BI consumes Gold tables or bounded exports, not raw source files |
| FR-18 | Document operational recovery | Runbook covers missing file, bad schema, data-quality failure, partial execution |

## 5.2 Non-functional requirements

| ID | Area | Target / expectation |
|---|---|---|
| NFR-01 | Data integrity | Critical acceptance and reconciliation invariants pass before publication |
| NFR-02 | Idempotency | Replay of identical input creates no change in published fact/aggregate counts or sums |
| NFR-03 | Observability | Every run has searchable status, source IDs, counts, durations, and failure details |
| NFR-04 | Recoverability | A controlled failure can be retried without corrupting a previously published version |
| NFR-05 | Reproducibility | Environment setup and versions pinned/documented; deterministic fixture-based tests |
| NFR-06 | Maintainability | Transformation modules small, documented, config-driven, and tested |
| NFR-07 | Cost | Target no-cost or local tooling; declare any cloud usage and respect quotas |
| NFR-08 | Scalability | Benchmark on one month and three months; show measured bottlenecks and choices |
| NFR-09 | Security | No credentials in Git; sourced data only; secrets use environment variables |
| NFR-10 | Auditability | Inputs, code version, ruleset version, and resulting published output version linked |

**Performance rule:** Do not fabricate records/sec or a speedup target. Measure on the actual machine or free cloud environment and report hardware/configuration, data volume, elapsed time, and limitations. Faster execution is desirable but correctness comes first.

---

# 6. Architecture and processing strategy

## 6.1 Logical architecture

```text
                 TLC monthly Parquet       TLC zone lookup
                        |                        |
                        v                        v
                  Python ingestion + source manifest
                        |
                        v
             BRONZE: immutable raw/versioned inputs
                        |
             contract checks + PySpark validation
                        |
                +-------+--------+
                |                |
                v                v
       SILVER: accepted      QUARANTINE:
          trip records      invalid rows/reasons
                |
            SQL/PySpark modeling
                |
                v
       GOLD: dim_zone, dim_date, fact_yellow_trip,
             agg_daily_zone, agg_monthly_kpi
                |
                v
       Power BI / documented metric queries

 Cross-cutting: orchestration | logs | data-quality reports
                tests | source lineage | Git/CI

 Optional independent branch: Open-Meteo -> weather staging ->
            daily weather aggregate -> Gold comparison dataset
```

## 6.2 Why Medallion remains appropriate

- **Bronze** isolates raw/versioned external records and supports inspection/replay.
- **Silver** applies explicit technical and business validation rules.
- **Gold** exposes dimensional and aggregated datasets intended for business analysis.

Kimball-style dimensional modeling lives **inside Gold**; it is not a separate mandatory layer after Gold.

## 6.3 Unit of incremental processing

The TLC source is a **monthly published file**, not a database change stream and does **not** guarantee a unique trip record ID. Therefore, build **file-level incremental batch ingestion** rather than claiming source-level CDC.

**Source-file identity:** `(source_type, source_month, file_sha256)` plus source URI and ingest timestamp.  
**Logical reporting partition:** `(source_type, source_month)` for MVP.  
**Same hash + already published:** skip/no-op.  
**New month:** ingest and publish that month.  
**Changed hash for published month:** stage and validate new version, then replace that month's Silver/Gold published data, keeping the prior published version available for rollback according to retention settings.

For the MVP, only trips whose **pickup calendar month matches the file's source month** qualify for the published fact. Out-of-file-month rows are explicitly quarantined or assigned an exclusion status with counts and reason. This conservative rule prevents cross-month contamination when publishing by file month. Revisit it only with an explicit impacted-partition rebuild design.

**Do not silently deduplicate entire trips using a hash of business columns.** Two different real trips can have identical visible field values. A diagnostic row fingerprint may flag *possible* duplicates, but it is not a trusted trip identifier.

## 6.4 Publishing safety

- Process candidate data in staging/versioned tables or paths.
- Pass schema, quality, and reconciliation gates **before** marking a source version published.
- Use an atomic table/partition replacement or version-pointer switch supported by the chosen execution engine; a multi-table Gold publication may need a controlled publish manifest so dashboards never combine incompatible versions.
- Mark publication success **after** required outputs are committed.
- On failure, preserve last successful published output and flag the candidate failed.
- Define how to rebuild Gold aggregates for the impacted month and verify against Silver.

---

# 7. Technology and environment decisions

## 7.1 Recommended minimal stack

| Component | Preferred MVP implementation | Rationale |
|---|---|---|
| Development | VS Code, Python virtual environment, Git | Reproducible and familiar |
| Source retrieval | Python (`requests` or `httpx`) | Robust download and error handling |
| Data processing | PySpark | Learn production-relevant distributed DataFrame patterns |
| Storage | Local Parquet + Delta Lake | Learn columnar files, transactions, versioned tables |
| Modeling | Spark SQL / PySpark SQL | Fact, dimensions, aggregates |
| Orchestration | Python command runner/DAG first, optional Databricks Jobs or Airflow later | Avoid setup complexity before reliable transforms |
| Quality and tests | Pytest + explicit Spark-based validation tests | Testable rules, positive and negative fixtures |
| CI | GitHub Actions | Automated lint/test on PR/push |
| Reporting | Power BI with bounded Gold exports or supported connector | Concrete consumer of the published model |
| Cloud exposure | Databricks Free Edition after local MVP | Valuable platform knowledge without making it a prerequisite |

**Practical limit:** Databricks Free Edition is quota-limited and serverless-only; outbound internet connections are restricted. Do not design the primary download step to depend on arbitrary external network access from its notebooks. Fetch files locally and stage/upload them through an allowed method if needed. A local-first pipeline is the baseline even if the cloud demonstration is postponed.

## 7.2 Development discipline

- Choose and document Python, Java, PySpark, Delta, and Spark versions **after** testing compatibility.
- Use an isolated environment and lock dependency versions.
- Keep credentials and large data out of Git.
- Set Spark session timezone deliberately, document source timestamp assumptions, and log the configuration.
- Partition and cache only when benchmarks justify them; no cargo-cult optimization.
- Validate all joins for cardinality and missing dimension keys.

---

# 8. Data contracts and transformation policies

## 8.1 Source data contract: yellow trips

**Critical fields required for analytical trip qualification** (source naming shown):

| Field | Logical role | Transformation rule |
|---|---|---|
| `tpep_pickup_datetime` | Trip event time | Parse as NYC-local source time; validate existence and month policy |
| `tpep_dropoff_datetime` | End time | Parse, require later than pickup for accepted duration-based analytics |
| `PULocationID` | Pickup zone key | Cast/validate; missing or unmatched -> defined unknown zone or quarantine based on rule |
| `DOLocationID` | Dropoff zone key | Same policy as pickup zone |
| `trip_distance` | Trip miles | Numeric; negative impossible -> quarantine; zero -> quality flag/review |
| `fare_amount` | Meter fare | Numeric; suspicious negative amounts flagged for review, not silently set to zero |
| `total_amount` | Passenger charge reported | Numeric; classify negative/zero thoughtfully, keep raw values for audit |
| `payment_type` | Payment classification | Map documented codes; unknown values remain explicit unknown |
| `passenger_count` | Driver-reported passengers | Numeric and nullable; zero or unusual values flagged according to policy |
| `VendorID`, `RatecodeID` | Source classifications | Preserve codes and map through documented code dictionaries |
| `cbd_congestion_fee` | 2025 congestion fee | Nullable or optional for earlier schema versions; never assume present historically |

Also preserve available `tip_amount`, `tolls_amount`, `extra`, `mta_tax`, `congestion_surcharge`, `airport_fee`, `improvement_surcharge`, `store_and_fwd_flag` and the original source columns.

**Schema change policy:**

1. New optional field: record difference, permit staged ingestion when safely compatible, update contract and tests before Gold exposure.
2. Missing critical field: fail contract gate and do not publish.
3. Unexpected type or renamed field: fail or handle with explicitly versioned mapping after inspecting the change.
4. Old years may have a different schema: do not claim one contract works for every year before verifying.

## 8.2 Required metadata and control tables

- `control.source_file_manifest`: source_type, source_month, source_uri, content_sha256, file_bytes, first_seen_at_utc, raw_path, schema_hash, latest_status.
- `control.pipeline_run`: run_id, requested_months, code_version, rules_version, started_utc, ended_utc, status, failure_reason.
- `control.run_step`: run_id, step_name, attempt, status, counts, elapsed_seconds, error_summary.
- `control.published_partition`: source_type, source_month, source_version, silver_version, gold_version, published_utc, run_id.
- `quality.rule_results`: run_id, source_version, rule_id, severity, failures, affected_rows, result.

Exact physical schemas can vary; the **semantics** must be preserved.

## 8.3 Bronze dataset

`bronze.yellow_taxi_raw` (or source-versioned raw files) should preserve every available record field and connect it to the source file version. Record raw schema, ingestion date, URI/hash, and row count. No destructive filling of nulls, no deduplication, no silently dropping outliers.

## 8.4 Silver datasets

- `silver.yellow_trip_accepted`: rows meeting publication rules; normalized column names; derived `trip_duration_minutes`; quality flags; provenance metadata.
- `silver.yellow_trip_quarantine`: original/parsed payload (or stable pointer to raw), `reason_codes`, failed rule severity, run and source version.
- `silver.taxi_zone`: validated zone lookup with unique `location_id`.
- `quality.file_reconciliation`: summary per source version with counts, percentages, thresholds, and reason distribution.

### Quality-rule catalog (initial proposal)

| Rule | Condition | Severity | Action |
|---|---|---|---|
| DQ-01 | Required pickup or dropoff timestamp null/unparseable | Critical | Quarantine |
| DQ-02 | Dropoff time not after pickup | Critical for duration/reporting | Quarantine |
| DQ-03 | Pickup month differs from file month | Critical for MVP partition contract | Quarantine or documented excluded |
| DQ-04 | Trip distance negative | Critical | Quarantine |
| DQ-05 | Pickup/dropoff zone missing or unknown | Review | Unknown-zone member or quarantine; decide via ADR |
| DQ-06 | Passenger count null/zero/unusually high | Warning | Preserve, flag, profile |
| DQ-07 | Trip distance zero or extreme | Warning | Preserve/flag unless separately justified exclusion |
| DQ-08 | Negative/zero fare or total charge | Warning/review | Preserve for financial reconciliation; analyze with business rule |
| DQ-09 | Unknown vendor/payment code | Warning | Map to Unknown and record frequency |
| DQ-10 | New/missing fields or incompatible source type | Contract gate | Block if critical; review additive changes |
| DQ-11 | Suspected duplicated business-field fingerprint | Informational | Count candidates; do not automatically delete |
| DQ-12 | Raw vs classification counts inconsistent | Critical | Block publication |

**Don't pretend every unusual value is wrong.** Negative financial values may be adjustments, and zero distances may reflect particular trip situations or imperfect reporting. Collect evidence before deciding whether to exclude. Review thresholds after profiling real months.

### Rule-versioning requirement

Record `rules_version`, effective date, and rationale for every rule change. Reprocessing historical data with a changed ruleset must explicitly identify impacted partitions and resulting KPI shifts.

## 8.5 Time handling policy

Source taxi timestamps are treated as **local New York wall-clock times as a documented working assumption**; inspect source documentation and actual values before committing. Store a local timestamp and pickup date. Fall daylight-saving repeated hours can be ambiguous without an original UTC offset. Do not invent a unique UTC instant for ambiguous local trip times.

For the MVP, weather is joined at the **local calendar date** grain, not directly at ambiguous local hourly timestamps. For a future hourly weather join, document UTC conversion, DST ambiguity handling, and join-count assertions.

---

# 9. Gold model and metric definitions

## 9.1 Gold star schema

**Core fact:** `gold.fact_yellow_trip`  
**Grain:** One **accepted TLC source trip record** from the current published version of its source month. This is a record grain, not a guarantee of unique real-world trips.  
**Source trip identifier:** None guaranteed. A `trip_fingerprint` may support diagnostics but cannot enforce uniqueness.

Suggested columns: `source_month`, `pickup_date_key`, `pickup_hour`, `pickup_zone_key`, `dropoff_zone_key`, `vendor_code`, `payment_type_code`, `trip_distance_miles`, `trip_duration_minutes`, `fare_amount_usd`, `total_amount_usd`, `tip_amount_usd`, `passenger_count`, `quality_flags`, `source_version`.

**Dimensions:**

- `gold.dim_zone`: `zone_key` (surrogate), `location_id` (natural), `borough`, `zone_name`, `service_zone`, `is_unknown`.
- `gold.dim_date`: `date_key` (YYYYMMDD), full_date, year, quarter, month, day_of_week, weekend flag.
- Optional `gold.dim_payment_type` and `gold.dim_vendor` if business logic warrants them; small codes may remain documented attributes in the fact for MVP.

**Aggregated data marts:**

- `gold.agg_daily_pickup_zone`: pickup_date, zone_key, accepted_trip_count, sum_total_amount, sum_fare_amount, avg_trip_distance, avg_trip_duration.
- `gold.agg_monthly_kpi`: source_month, accepted_trip_count, total_recorded_charges, avg_fare, avg_duration, null/unknown breakdown when relevant.
- Optional `gold.agg_daily_weather_demand`: pickup_date, recorded trips, precipitation, temperature; explicitly **daily**, not hourly.

### Important modeling decisions

- The pickup zone and dropoff zone are **role-playing references** to the same zone dimension.
- Use a documented unknown zone row for recoverable missing zone IDs, or quarantine consistently; never drop fact rows due to an accidental inner join.
- Zone names represent reference information at a point in time. Start with Type 1 lookup management; do not invent customer history or SCD Type 2 for a dataset without customer identities.
- If Gold models are rebuilt from changed file versions, keep published version metadata so totals can be audited.

## 9.2 Formal business metrics

| KPI | Definition | Caveat |
|---|---|---|
| Accepted trips | `COUNT(*)` in accepted fact population | Not guaranteed unique physical rides |
| Recorded passenger charges | `SUM(total_amount_usd)` over qualifying records | Not net revenue or cash tips |
| Average fare | `AVG(fare_amount_usd)` over non-null qualifying rows | Provide denominator/missing-value policy |
| Average duration | `AVG(trip_duration_minutes)` | Based on accepted positive-duration trips |
| Average distance | `AVG(trip_distance_miles)` | Preserve zero values unless policy excludes |
| Trips by pickup zone | Count accepted fact rows grouped by pickup zone | Zone join must preserve row counts |
| Quarantine rate | Quarantined source rows / raw source rows | Report per file and rule version |

Avoid calling raw `total_amount` “revenue” without clarifying the definition. Add monthly totals for all components only if the aggregation is meaningful and correctly reconciled; source financial components may not always sum under every record condition.

## 9.3 Reconciliation tests

For every current source-month publication:

1. `raw_count = accepted_count + quarantine_count + excluded_count` (mutually exclusive classification; exclude count may be zero).
2. `gold_fact_count = silver_accepted_count` for eligible population.
3. `sum(gold.fact.total_amount) = sum(silver.accepted.total_amount)` within documented numeric precision.
4. `sum(gold.agg_daily_pickup_zone.accepted_trip_count) = gold_fact_count`.
5. Every fact's role-playing zone keys resolve to one zone dimension row (including Unknown if used).
6. Latest published source version matches control manifest and run metadata.

---

# 10. Weather enrichment (second milestone, not MVP blocker)

**Source:** Open-Meteo Historical Weather API `https://archive-api.open-meteo.com/v1/archive`, selecting a documented NYC coordinate, date range, and variables such as `temperature_2m`, `precipitation`, and `rain` where available.

**Design:** Retain weather API request parameters and returned dataset metadata. Store raw JSON or an immutable extract. Process into `silver.weather_hourly`, normalize to NYC local dates with documented DST logic, then aggregate to `gold.dim_weather_day` or `gold.agg_weather_daily` with **one row per city/date** before joining to daily trip volume.

**Critical safeguards:**

- Do not join one trip to multiple hourly weather rows and accidentally inflate trip counts.
- Do not claim weather measurements represent conditions in every taxi zone; a single grid cell approximates an area.
- Reanalysis/weather-model data is not the same as a direct physical observation at every trip time.
- Clearly label the analysis correlational, not causal.
- Credit Open-Meteo in the dashboard and README, following source terms.

**Acceptance:** The daily weather table has one row per local date, weather joins do not change the original trip count, and at least one documented rain/no-rain comparison is reproducible.

---

# 11. Orchestration, logging, failure recovery

## 11.1 Pipeline task graph

`discover -> download -> fingerprint -> contract_check -> bronze_load -> silver_transform -> dq_validate -> gold_stage -> reconcile -> publish -> report`

Zone reference ingestion can be a prerequisite task. Weather is a separate dependency only for weather-enriched aggregates; failure to retrieve weather should not invalidate core yellow-taxi Gold data.

## 11.2 State machine

`DISCOVERED -> DOWNLOADED -> VALIDATED -> PROCESSING -> READY_TO_PUBLISH -> PUBLISHED`  
Exceptional paths: `SKIPPED_UNCHANGED`, `FAILED_DOWNLOAD`, `FAILED_CONTRACT`, `FAILED_QUALITY`, `FAILED_COMPUTE`, `FAILED_PUBLISH`.

State changes must be persisted; a failed run is not treated as success merely because some staged tables exist. Each attempt uses a stable run ID and source version identifiers.

## 11.3 Run metadata and logging

At minimum emit structured records with `timestamp_utc`, `run_id`, `source_type`, `source_month`, `source_sha256`, `step`, `attempt`, `status`, `elapsed_ms`, `row_counts`, and meaningful error codes. Provide aggregate operational metrics: files processed/skipped, success/failure count, quarantine rate, duration by step, and last publish time.

## 11.4 Recovery runbook scenarios

| Failure | Expected response | Proof |
|---|---|---|
| Source URL unavailable | Retry transient failures with capped backoff; fail clearly if persistent | Simulated download timeout |
| Source schema missing critical column | Block publish and save contract diff | Fixture with missing pickup time |
| Unexpected values / severe quality issue | Quarantine individual rows; block if reconciliation fails | Corrupt fixture with rule report |
| Pipeline crashes after Silver staging | Prior Gold remains published; restart safely | Terminate controlled test run |
| Same source file replayed | Skip or no-op with log entry | Compare facts before/after |
| Published month's file changes | Stage new hash, rebuild impacted partitions, publish new version | Controlled local modified-copy scenario |
| Weather API fails | Core pipeline still succeeds; weather report marked stale/unavailable | Mock API 503 |
| Quality rule changes | Version rules and backfill specified months | Compare before/after reports |

**Note:** Modified-copy tests are controlled fault-injection tests, not evidence that TLC published a specific corrected file in that scenario.

---

# 12. Testing strategy and validation matrix

## 12.1 Test levels

**Unit tests:** parsing, contract validation, code mappings, date rules, reason-code assignment, manifest state transitions.  
**Spark transformation tests:** tiny real-shape fixture DataFrames; expected accept/quarantine outputs.  
**Integration tests:** small downloaded or curated Parquet sample from one official month through Gold.  
**Pipeline/recovery tests:** repeated runs, changed file hash, partial failure, rerun, and month replacement.  
**Analytics tests:** join cardinality, Gold-to-Silver count and sum reconciliation, metric definitions.  
**CI checks:** formatting/linting, unit tests, integration smoke test where runner resources allow.

## 12.2 Mandatory test cases

| Test ID | Given | Expected |
|---|---|---|
| T-01 | Valid file fixture | Correct raw/accepted counts; Gold available |
| T-02 | Identical file rerun | No additional published records or charges |
| T-03 | Changed monthly file bytes | New source version logged; controlled month rebuild |
| T-04 | Missing pickup timestamp field | Contract gate fails; existing published partition unchanged |
| T-05 | Row with dropoff <= pickup | Row quarantined; counted in reason report |
| T-06 | Negative distance | Row quarantined; raw preserved |
| T-07 | Null passenger count | Rule-specific flag, not silent loss |
| T-08 | Duplicate-looking business values | Diagnostic count only; no unsupported dedup |
| T-09 | Dimension unmatched zone | Unknown/quarantine policy applied and count reconciles |
| T-10 | Staged compute failure | Run FAILED; last published result unchanged |
| T-11 | Bad Gold join | Test catches increased record count |
| T-12 | Changed rule set | Impacted month identified, rules version retained |
| T-13 | Month-boundary source record | Explicit quarantine/exclusion; no cross-month drift |
| T-14 | Weather date with DST change | Daily aggregate join remains one-to-one |
| T-15 | Multi-month backfill | End-state counts match independent reference build |

## 12.3 Evidence policy

For each mandatory test preserve: test name, code link, fixture provenance, command, pass/fail, and output/log screenshot or machine-readable report. Never present tests as passed before running them.

---

# 13. Implementation plan and phase gates

The phases are designed to teach **one new concept at a time**. Do not move to the next phase until its exit conditions pass. A realistic personal plan might spread this over **8–12 weeks depending on available hours and debugging**, but there is no deadline requirement.

| Phase | Learn | Build tasks | Required exit evidence |
|---|---|---|---|
| P0: Foundation | Business requirements, Git, source docs, Parquet | Repo, ADRs, setup, download one file, source profiling notebook | Written data profile and first ADR |
| P1: Ingestion | Python I/O, HTTP failures, hashing, config | Downloader, manifest, source month CLI, raw staging | Two-run identical-file skip test |
| P2: Spark/Delta Bronze | PySpark DataFrames, schemas, Delta transactions | Raw schema discovery, versioned Bronze, lineage fields | Source vs Bronze reconciliation |
| P3: Silver/data quality | Spark transforms, validation, quarantine | Contract checks, rules, accepted/quarantine, DQ report | Explicit rule fixture tests |
| P4: Incremental & replay | Idempotency, file versioning, backfill | Month-level publication, modified-file replacement, rollback drill | Same-file and changed-file demos |
| P5: Gold/modeling | Grain, star schema, analytic metrics | Zone/date dimensions, fact, aggregates, reconciliation | Passed no-fanout and KPI tests |
| P6: Operations | Orchestration, failure policies, observability | DAG/runner, logs, retries, runbook, CI | Controlled failure + recovery video/log |
| P7: Reporting | Power BI, stakeholder communication | Dashboard, source lineage and quality page | Reproducible business questions |
| P8: Extension & portfolio | Weather, cloud, benchmarks, writing | Weather join, cloud demo if feasible, perf comparison, README | Published technical case study |

## 13.1 Detailed task backlog

### P0 — Architecture and real-data investigation
- [ ] Read the TLC trip-data landing page and yellow-taxi data dictionary.
- [ ] Create repository and standard folders; add `.gitignore`, `README.md`, environment setup.
- [ ] Download January 2025 yellow trips and taxi-zone lookup from the official website.
- [ ] Inspect exact Parquet schema, physical types, null percentages, min/max times, distance/fare distributions, pickup-month mismatch count, payment codes, and candidate duplicate fingerprints.
- [ ] Write `docs/source_profile_2025_01.md` with **actual results** and at least five justified data-quality decisions.
- [ ] Draft ADR-001 (why file-level replacement rather than row-level CDC).
- [ ] Confirm working Spark/Delta version compatibility locally before building dependent code.

### P1 — Python ingestion and tracking
- [ ] Implement month selection via CLI/config.
- [ ] Build source-link resolver and downloader with retries and temporary-file atomic rename.
- [ ] Calculate SHA-256 and file size; register manifest entry.
- [ ] Skip identical successfully published sources; do not confuse downloaded with published.
- [ ] Record meaningful errors, timeouts, and interrupted-download cleanup.
- [ ] Unit-test month selection and manifest transition rules.

### P2 — Bronze
- [ ] Read the actual Parquet source with PySpark.
- [ ] Persist raw/versioned records and raw schema metadata.
- [ ] Attach file/run lineage without overwriting source business fields.
- [ ] Assert source file read count equals registered Bronze count.
- [ ] Verify a replay doesn't create a duplicate active Bronze publication.

### P3 — Silver and data quality
- [ ] Write source contract definition with required/optional fields.
- [ ] Normalize names/types and calculate trip duration.
- [ ] Build accepted/quarantine classification with explicit reason codes.
- [ ] Generate DQ summary per file and severity.
- [ ] Review zero/negative monetary values and unknown locations using actual profile evidence.
- [ ] Add Spark fixture tests for critical rules.

### P4 — Safe incremental updates
- [ ] Process January, then February and March without reloading previously published months.
- [ ] Add month/version-aware publication staging.
- [ ] Create a controlled modified-copy of a month's source and demonstrate replacement.
- [ ] Verify identical rerun produces identical published monthly counts and sums.
- [ ] Intentionally fail mid-run and demonstrate prior published results remain queryable.
- [ ] Build CLI option for selected-month backfill/rebuild and a rollback procedure.

### P5 — Gold modeling and analytics
- [ ] Define fact grain and roles of pickup/dropoff zone keys.
- [ ] Validate zone lookup uniqueness before creating `dim_zone`.
- [ ] Implement `dim_date`, `fact_yellow_trip`, daily zone aggregate, monthly KPI aggregate.
- [ ] Write metric dictionary and SQL reconciliation tests.
- [ ] Demonstrate no record multiplication in Gold joins.

### P6 — Orchestration and operations
- [ ] Create declarative pipeline order with error propagation.
- [ ] Add structured run/step logging and persisted status.
- [ ] Decide retryable vs non-retryable errors.
- [ ] Add pytest and a GitHub Actions test workflow.
- [ ] Write operator runbook: discover, run, inspect, retry, rebuild, rollback.
- [ ] Record a failure/recovery walkthrough with real logs.

### P7 — Analytics output
- [ ] Build Power BI report using Gold data or bounded export.
- [ ] Create pages: Demand Overview; Zone Performance; Data Quality/Refresh Status.
- [ ] Document charge/fare terminology and filtering rules.
- [ ] Cross-check displayed numbers against reconciled SQL queries.
- [ ] Save 3–5 screenshots and concise business interpretation, not just visual screenshots.

### P8 — Optional depth and publication
- [ ] Retrieve weather history with request metadata and attribution.
- [ ] Build one-row-per-day weather aggregate and safe daily demand comparison.
- [ ] Benchmark one vs three months; record Spark settings and test conditions.
- [ ] Run a cloud notebook/job demonstration within Databricks Free Edition restrictions if feasible.
- [ ] Publish architecture diagram, ADRs, test evidence, benchmark table, sample demo and final README.

---

# 14. Proposed repository organization

```text
urban-mobility-intelligence/
├── README.md
├── pyproject.toml               # or requirements files, pinned
├── .gitignore
├── .github/workflows/ci.yml
├── configs/
│   ├── pipeline.yaml
│   └── source_contracts/yellow_taxi_2025.yaml
├── docs/
│   ├── PROJECT_SPEC.md
│   ├── architecture.md
│   ├── business_metrics.md
│   ├── data_dictionary.md
│   ├── quality_rules.md
│   ├── source_profile_2025_01.md
│   ├── operations_runbook.md
│   ├── test_plan.md
│   ├── performance_benchmarks.md
│   ├── demo_script.md
│   └── adr/
│       ├── 001-file-level-incremental.md
│       ├── 002-monthly-publish-strategy.md
│       └── 003-unknown-zone-policy.md
├── src/urban_mobility/
│   ├── cli.py
│   ├── config.py
│   ├── ingestion/{sources.py, downloader.py, manifest.py}
│   ├── bronze/{load.py, metadata.py}
│   ├── silver/{transform.py, contracts.py, classify.py}
│   ├── gold/{dimensions.py, facts.py, aggregates.py}
│   ├── quality/{rules.py, reconciliation.py}
│   ├── orchestration/{pipeline.py, state.py}
│   └── observability/{logging.py, metrics.py}
├── sql/gold/
│   ├── metric_definitions.sql
│   └── reconciliation.sql
├── tests/
│   ├── unit/
│   ├── spark/
│   ├── integration/
│   └── fixtures/
├── notebooks/                    # exploration only; core logic stays in src/
├── dashboards/                   # PBIX and exported screenshots, as appropriate
├── scripts/                      # setup, sample run, demo fault injection
└── data/                         # ignored except tiny licensed fixtures
```

**Repository rule:** Exploratory notebooks are welcome, but the core pipeline should live in importable, testable modules. Avoid a single notebook containing all business logic.

---

# 15. Documentation artifacts to maintain during development

| Artifact | Purpose | Update event |
|---|---|---|
| `README.md` | Problem, features, setup, demo, results | Every completed milestone |
| `docs/architecture.md` | Diagram, components, flow, trade-offs | Architecture change |
| `docs/source_profile_2025_01.md` | True observations from source data | Initial analysis / new source year |
| `docs/data_dictionary.md` | Column definitions and lineage | Schema/metric change |
| `docs/quality_rules.md` | Rule definitions, decisions, examples | Rule change |
| `docs/business_metrics.md` | KPI grain, numerator, denominator, filters | Metric change |
| `docs/adr/*.md` | Alternatives, decisions, consequences | Any consequential design decision |
| `docs/operations_runbook.md` | Routine execution and incident recovery | Recovery procedure change |
| `docs/test_plan.md` | Requirement-to-test traceability | Test additions |
| `docs/performance_benchmarks.md` | Actual measured performance | Benchmark run |
| `docs/demo_script.md` | Short reviewer/interviewer walkthrough | Feature completion |

## 15.1 Architecture decision record template

```text
ADR-###: <decision>
Status: proposed | accepted | superseded
Context: What constraint or business problem triggered this?
Options: What realistic alternatives were evaluated?
Decision: What was selected and why?
Consequences: What gets easier/harder?
Validation: What test or benchmark will verify the choice?
```

## 15.2 Dataset-profiling template

```text
Source month:
Source URI and hash:
Physical Parquet schema:
Raw row count:
Column null rates:
Min/max pickup and dropoff timestamps:
Trips outside nominal source month:
Distance and charge distributions:
Zone lookup mismatch count:
Candidate duplicate fingerprint count (not proof of duplicates):
Unexpected categories/types:
Proposed DQ policies and rationale:
Open questions:
```

---

# 16. Deliverables and interview-ready evidence

## 16.1 Required final deliverables

1. Public source-code repository, with useful README and no bulky downloaded dataset committed.
2. Architecture image/diagram and three justified ADRs.
3. Working repeatable 3-month sample pipeline, plus automated tests.
4. Source manifest, quality report, reconciled Gold model, and logs.
5. Failure-injection and replay evidence (before/after counts and status).
6. Dashboard/report and reconciled KPI definitions.
7. Reproducibility guide, versions, known limitations, and benchmark measurements.
8. Short demo (3–5 minutes) and one-page case study summarizing outcomes.

## 16.2 Suggested demo flow

- **0:00–0:30:** Business problem, authentic source, high-level design.
- **0:30–1:15:** Show input file metadata and source profiling findings.
- **1:15–2:00:** Run/simulate a normal ingestion; show Bronze, quality report, Gold counts.
- **2:00–3:00:** Replay same file (no duplicate) and show controlled failed run/recovery.
- **3:00–4:00:** Show SQL reconciliation, simple dashboard, and one architectural trade-off.
- **4:00–5:00:** Discuss measured performance, limits, and next improvement.

## 16.3 Technical interview questions you must be able to answer

- Why did you choose file-level incremental processing over row-level CDC?
- How is a rerun different from a backfill or source-file revision?
- How do you prove that failed processing did not corrupt the published result?
- How did you determine which anomalous records to quarantine and which to flag?
- Why is the Gold fact table at that grain? What happens if zones are not unique?
- What does Delta Lake add compared with raw Parquet files?
- How do you prevent a weather join from inflating trip counts?
- What did you measure when optimizing processing, and what did not improve?
- What would change for a true event stream or an operational database with CDC?
- What would you redesign if the platform needed daily SLA-backed production deployment?

---

# 17. Project risk register

| Risk | Likelihood / impact | Mitigation |
|---|---|---|
| Spark/Java/Delta package incompatibility | Medium / High | Pin tested versions in P0; small smoke test first |
| Unbounded data volume on personal hardware | High / Medium | One-month start, then 3 months, then measured scaling |
| TLC schema drift | Medium / High | Source contracts and version-specific mappings |
| No unique trip ID | Certain / High | File-version snapshots and month-level replacement; no fake CDC claims |
| Misleading financial semantics | Medium / High | TLC data dictionary and explicit KPI definitions |
| Join fan-out | Medium / High | Uniqueness and reconciliation tests before publication |
| Free cloud resource restrictions | High / Medium | Local-first architecture; cloud only optional |
| Weather timezone ambiguity | Medium / Medium | Daily-grain enrichment first; DST-aware extension only after tests |
| Scope creep | High / High | Keep optional extensions out of MVP gate |
| Project looks like another tutorial | Medium / High | ADRs, real profiling, recovery evidence, measurable benchmark, reproducibility |

---

# 18. Definition of done and release checklist

### Engineering completion
- [ ] FR-01 through FR-18 have implementations or explicitly documented justified deviations.
- [ ] All MVP acceptance checks in Section 3.2 pass.
- [ ] All mandatory test cases T-01 through T-15 applicable to MVP pass; optional weather test can be deferred with status noted.
- [ ] No silently dropped records; source-to-published counts reconcile.
- [ ] Rerun, modified file, deliberate failure, and recovery demos are reproducible.
- [ ] Documented limitations are accurate and visible.

### Portfolio completion
- [ ] Public README links to architecture, data sources, setup, screenshots, test instructions, and demo.
- [ ] At least three meaningful ADRs document trade-offs.
- [ ] At least one measured benchmark table is reproducible.
- [ ] Resume bullets reference only implemented and verified features.
- [ ] A hiring manager can understand the project in under two minutes from the README.

### Honest claim boundaries

Do **not** claim live production deployment, commercial-scale SLAs, millions of messages per second, real CDC, strong exactly-once delivery across all external systems, or verified performance improvements without measured proof. This is a portfolio implementation of production-oriented techniques.

---

# 19. Immediate start guide: first five working sessions

**Session 1 — Understand the source**
1. Read the TLC landing page and Yellow Taxi data dictionary.
2. Create a Git repository with this spec in `docs/PROJECT_SPEC.md`.
3. Download the January 2025 yellow trip Parquet file and zone lookup; save the original file without editing it.
4. Write down the exact downloaded file names and source URLs.
5. Deliverable: `docs/data_sources.md` and a committed project structure.

**Session 2 — Profile the real dataset**
1. Open Parquet with PySpark (or use DuckDB/Pandas *only for initial inspection* if Spark setup is pending).
2. Print schema, row count, and null counts.
3. Measure invalid timestamps, negative/zero distances, out-of-month pickups, and unknown zone values.
4. Save findings, query snippets, and unexpected cases in `docs/source_profile_2025_01.md`.
5. Deliverable: real observations, not fabricated sample findings.

**Session 3 — Design the contract**
1. Classify source columns as required, optional, or informational.
2. Decide type conversions and rule severity using the profile.
3. Write ADR-001 on file-level incremental strategy and no unique trip ID.
4. Define exact raw/accepted/quarantined count reconciliation.
5. Deliverable: contract YAML, DQ rule table, and accepted ADR.

**Session 4 — Build minimum ingestion**
1. Implement a Python CLI selecting January 2025.
2. Download to a temporary file, verify bytes and SHA-256, and move to final raw location only on success.
3. Write/inspect manifest entry; rerun to demonstrate unchanged-file detection.
4. Deliverable: tested, logged ingestion command.

**Session 5 — Bronze smoke test**
1. Read the downloaded Parquet file in a minimal PySpark script.
2. Write versioned Bronze data and record source metadata.
3. Assert source and Bronze counts agree.
4. Deliberately break one input path and confirm helpful logging rather than silent success.
5. Deliverable: first end-to-end ingestion-to-Bronze proof and green tests.

**First checkpoint:** Stop before Silver until the manifest, raw preservation, environment, and initial profile can be explained confidently.

---

# 20. Reference materials and attribution

1. NYC Taxi & Limousine Commission, **TLC Trip Record Data** — https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page  
   Official monthly file links and published caveats.
2. NYC TLC, **Yellow Taxi Trip Records Data Dictionary** (March 18, 2025) — https://www.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf  
   Official field semantics, code meanings, and the cash-tip limitation.
3. NYC TLC, **Taxi Zone Lookup CSV** — https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv  
   Location-to-zone mapping; inspect actual fields on download.
4. Open-Meteo, **Historical Weather API** — https://open-meteo.com/en/docs/historical-weather-api  
   Optional historical weather enrichment and timestamp behavior.
5. Open-Meteo, **Terms / License** — https://open-meteo.com/ and https://github.com/open-meteo/open-meteo  
   Non-commercial API usage conditions and CC BY 4.0 data attribution.
6. Delta Lake, **Official Documentation** — https://docs.delta.io/  
   Transactional lakehouse tables, schema enforcement, updates, and versioning.
7. Databricks, **Free Edition Limitations** — https://docs.databricks.com/aws/en/getting-started/free-edition-limitations  
   Quotas, serverless compute, and restricted outbound network access.

**Document change log:** v1.0 — Initial full specification and recommended learning plan. Update this log whenever scope, data contract, or architecture changes.
