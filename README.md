# Redshift ↔ Snowflake Bidirectional Integration

A fully tested, serverless pattern for bidirectional data sync between AWS Redshift and
Snowflake using S3 as the interchange layer. No Airflow, Step Functions, or third-party
ETL tools required.

## What This Is

This project demonstrates how to move data reliably in both directions between Redshift
and Snowflake on a weekly (or configurable) schedule, using only native AWS and Snowflake
capabilities.

**Redshift → Snowflake:** A Redshift stored procedure tracks a watermark and UNLOADs only
changed rows to S3 as Parquet. Snowpipe auto-ingests the files into a Snowflake staging
table. A MERGE with deduplication upserts the delta into the Snowflake target table.

**Snowflake → Redshift:** A Snowflake Stream captures inserts, updates, and deletes on the
source table. A Snowflake Task exports the CDC delta to S3 as Parquet. A scheduled Redshift
COPY loads it into a staging table, and a DELETE+INSERT pattern applies the changes to the
target table — keeping Redshift as an exact mirror of Snowflake.

## Scale

- 15 Redshift source tables, ~10M rows each — bulk loaded once, incremental thereafter
- 1 Snowflake source table, ~2M rows — bulk loaded once, CDC incremental thereafter

## Key Design Decisions

- **Watermark-based incremental on Redshift side** — only changed rows exported after bulk load
- **Snowflake Streams for CDC** — captures inserts, updates, and deletes natively
- **Timestamped S3 paths** — prevents Snowpipe from skipping files on repeat runs
- **Staging tables** — raw CDC Parquet lands in staging; target tables stay clean
- **No external orchestration** — Redshift scheduled queries and Snowflake Tasks handle timing

## Files

| File | Description |
|---|---|
| `data_flow_runbook.md` | Full implementation guide with SQL, IAM config, setup steps, and known issues |
| `redshift_stored_procedure.sql` | Watermark table, incremental UNLOAD procedure |
| `snowflake_stream_task.sql` | Storage integration, stage, Snowpipe, stream, task, MERGE |
| `architecture_diagram.png` | Architecture diagram for both flows |
| `supply_chain_runbook.md` | Internal reference — supply chain dataset used for end-to-end testing |
| `supply_chain_runbook_customer.md` | Earlier draft of customer runbook (superseded by `data_flow_runbook.md`) |
