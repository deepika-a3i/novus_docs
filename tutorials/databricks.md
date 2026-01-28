# Databricks and Spark Tutorial

This guide explains how to run Novus pipelines in Databricks or Spark-enabled environments.

## When to use

- You want to write results into Delta tables.
- You are orchestrating jobs from notebooks and need a Spark session.
- You want to run extraction or VOC pipelines on DBFS paths.

## Use the run helper

The CLI exposes a programmatic helper that accepts a Spark session and command name:

```python
from novus_calls.cli.commands.run import run_with_spark

run_with_spark(
    spark=spark,
    command="voc-extract",
    config="/dbfs/config/voc_pipeline.yml",
    max_files=50,
    databricks=True,
    target_date="2025-10-20",
)
```

Supported commands include:

- `transcribe`
- `extract`
- `report`
- `build`
- `voc-extract`

## Databricks credentials

Set these environment variables (often via secret scope):

- `DATABRICKS_SERVER_HOSTNAME`
- `DATABRICKS_ACCESS_TOKEN`
- `DATABRICKS_WAREHOUSE_ID`

## Databricks configuration

Both the agent scorecard and VOC configs include a Databricks block or data source section. Ensure these match the target Delta table schema and partitioning before enabling upserts.

## Tips

- Test locally with `--no-databricks` before enabling MERGE.
- Keep `final_columns.yml` and the Delta table schema aligned.
- Use date-partitioned paths to avoid mixing runs.
