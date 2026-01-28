# Database and Storage

Database integration is primarily used for Databricks Delta tables. It is optional and can be disabled for local runs.

## Key modules

- `src/novus_calls/core/database/databricks.py`: handles Delta table writes and MERGE logic.
- `src/novus_calls/core/database/schemas.py`: schema helpers for column definitions.
- `src/novus_calls/core/database/utils.py`: utilities for Spark and table operations.

## How Databricks is used

- VOC pipeline: optional upsert into Delta tables after combining features.
- Scorecard pipeline: optional table writes depending on configuration.

## Configuration

Databricks configuration is set in YAML files:

- `config/voc_config/voc_pipeline.yml` under the `databricks` block.
- `config/agent_scorecard_config/pipeline.yaml` under `REPORT` or data source sections.

## Environment variables

Set these variables for Databricks connectivity:

- `DATABRICKS_SERVER_HOSTNAME`
- `DATABRICKS_ACCESS_TOKEN`
- `DATABRICKS_WAREHOUSE_ID`

Use `--no-databricks` flags during local tests to skip writes.
