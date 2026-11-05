# January 2025 NYC yellow-taxi source profile

**Status: Not analyzed yet.** Replace prompts and `TBD` only after inspecting the actual file.

| Observation | Actual value / note |
|---|---|
| Official file URL | https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2025-01.parquet |
| Download date/time (UTC) | 25/10/2026 |
| File size in bytes | 57772 |
| SHA-256 | 9AF277E4C0D3F9DEB30644DA822981E1E7DF6AF58313170FD3AA8A474485488A |
| Parquet row count | TBD |
| Physical schema and column types | TBD |
| Null rates by column | TBD |
| Min/max pickup and dropoff timestamps | TBD |
| Pickup timestamps outside 2025-01 | TBD |
| Distance and total amount distributions | TBD |
| Zone lookup unmatched IDs | TBD |
| Candidate duplicate field patterns (not real trip IDs) | TBD |
| Unrecognized payment/vendor codes | TBD |
| Source quality caveats | TLC states records not guaranteed accurate/complete |

## Proposed quality rules — justify with observed evidence

1. Rule and reason:
2. Rule and reason:
3. Rule and reason:
4. Rule and reason:
5. Rule and reason:

## Queries, code and evidence

- Processing environment/version: TBD
- Commands used: TBD
- Log/summary output committed (not raw data): TBD

## Open questions

- What should happen to trips with pickup times outside the nominal monthly file?
- What is the acceptable policy for missing or unmatched pickup/dropoff zone IDs?
- Are negative charge fields corrections or unacceptable records for certain metrics?
