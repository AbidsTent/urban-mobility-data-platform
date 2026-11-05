# Local data directory — intentionally excluded from Git

The directories/files under `data/` are local working inputs/outputs and are ignored by `.gitignore` (except this README).

**Initial official inputs:**

- Yellow-taxi Jan 2025: https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2025-01.parquet
- Zone lookup: https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv
- Landing page: https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

We will start download and source inspection in P0-02. Do not invent profiling numbers or commit monthly Parquet files to GitHub.
