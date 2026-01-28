# VOC Pipeline Tutorial

This tutorial explains how to run the Voice of Customer (VOC) pipeline and how the conditional extraction flow works.

## Inputs and configs

Primary config pack: `config/voc_config/`

Key files:

- `voc_pipeline.yml`: pipeline steps, models, routing, input/output.
- `fields/`: schema definitions for structured and list extractions.
- `prompts/`: system and user prompt templates.
- `field_mappings.yml`, `aggregation_rules.yml`, `final_columns.yml`: table combination logic.

Production configuration should be deployed to your target environment (e.g., `/dbfs/config/` for Databricks).

## Step 1: Create a local config

```bash
cp config/voc_config/voc_pipeline.yml config/local_voc.yml
```

Edit the paths:

- `base_config_folder`: `config/voc_config`
- `input.transcript_path`: `<transcripts-dir>/`
- `output.raw_features_path`: `<output-dir>/voc_features/raw/`
- `output.combined_table_path`: `<output-dir>/voc_features/combined/voc_combined.xlsx`

## Step 2: Run a small batch

```bash
novus voc-extract --config config/local_voc.yml --max-files 5 --no-databricks
```

## Step 3: Inspect outputs

- Raw JSON per step: `<output-dir>/voc_features/raw/`
- Combined table: `<output-dir>/voc_features/combined/voc_combined.xlsx`

## Date-based paths

VOC supports date resolution via:

```bash
novus voc-extract --config config/local_voc.yml --target-date 2025-10-20 --date-path
```

## How the routing works

The pipeline starts with `initial_screening`, then conditionally runs detailed steps based on extracted flags. Each detail step has its own schema and prompt templates. The combiner merges everything into a single row per call.

## Enable Databricks

Remove `--no-databricks` and ensure Databricks env variables are set. The pipeline will MERGE into the configured Delta table.

For schema-level details, see `configuration/voc-pipeline-config.md`.
