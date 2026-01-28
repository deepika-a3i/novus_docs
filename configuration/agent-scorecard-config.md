# Agent Scorecard Configuration Reference

This document describes the current agent scorecard configuration as implemented in `config/agent_scorecard_config/pipeline.yaml` and consumed by the CLI.

It is intended to be a practical reference for:

- Running extraction and reporting.
- Understanding what each field controls.
- Safely editing the pipeline without breaking downstream steps.

## File layout

Primary folder: `config/agent_scorecard_config/`

Key files:

- `pipeline.yaml`: top-level pipeline config (extraction + report settings).
- `extraction_pipeline.yaml`: extraction flow (steps and CSV wiring).
- `features.csv`: CSV catalog of features.
- `csv_questions/`: shared prompts + function schema for CSV features.
- `summary/`: prompts for call summaries.

Production configuration should be deployed to your target environment (e.g., `/dbfs/config/` for Databricks).

## Top-level fields

```yaml
USER_ID: "agent_scorecard"
DESCRIPTION: "Agent Scorecard pipeline"
USECASE: "renewals"
DEFAULT_LANGUAGE_MODEL: "gpt-4.1-mini"
```

- `USECASE` must be `renewals` or `sales`.
- `DEFAULT_LANGUAGE_MODEL` is used when steps do not specify their own model.

## Extraction section

```yaml
EXTRACTION:
  EXTRACTION_PIPELINE_CONFIGURATION: "./config/agent_scorecard_config/extraction_pipeline.yaml"
  TRANSCRIPT_PATH: "azure://novuscalls/adhoc/transcripts/"
  LANGUAGE_MODEL_NAME: "gpt-4.1-mini"
  LANGUAGE_MODEL_SEED: 0
  LANGUAGE_MODEL_TEMP: 0.0
  METADATA_GT: "<config-dir>/metadata_file.xlsx"
  OUTPUT_PATH: "<output-dir>/agent_scorecard/2025/12/11"
  OUTPUT_OVERWRITE: true
```

Key fields:

- `EXTRACTION_PIPELINE_CONFIGURATION`: the YAML used by `novus build`.
- `TRANSCRIPT_PATH`: folder or URI containing transcript JSON.
- `LANGUAGE_MODEL_*`: extraction model settings for the pipeline.
- `METADATA_GT`: optional metadata file used for conditional routing.
- `OUTPUT_PATH`: where raw extraction JSON and feature table are written.
- `OUTPUT_OVERWRITE`: set to false to skip files that already exist.

### Output behavior

`novus extract` writes per-call JSON files into `OUTPUT_PATH`, then generates:

- `feature_wide_table.xlsx`

## Report section

```yaml
REPORT:
  FEATURE_WIDE_TABLE_PATH: "azure://novuscalls/output/agent_scorecard/2025/10/10/feature_wide_table.xlsx"
  LLM_CONFIG:
    LANGUAGE_MODEL_NAME: "gpt-4.1-mini"
    LANGUAGE_MODEL_SEED: 0
    LANGUAGE_MODEL_TEMP: 0.0

  AGENT_DETAILS_PATH: "<config-dir>/agent_details.csv"
```

### Email configuration

The report service expects an `EMAIL` block in the report config:

```yaml
REPORT:
  EMAIL:
    SCORECARD_CONFIG_PATH: "<config-dir>/scorecard_email_config.yaml"
```

If this block is missing, scorecard emails will be skipped. Some legacy configs still contain `EMAIL_CONFIG_PATH` at the report root; prefer the `EMAIL.SCORECARD_CONFIG_PATH` format used by the current code.

### Field mapping and scoring

```yaml
REPORT:
  FIELD_MAPPING:
    agent_id_field: "agent_id"
    agent_name_field: "agent_name"
    call_id_field: "call_id"
    FEATURES:
      - feature_name: "personalized_pitch"
        display_name: "Personalized Pitch"
        description: "personalizing the pitch to the customer"
        weight: 0.20
        display_summary: true
```

- `feature_name` must match the `abbrev` in `features.csv`.
- `weight` values determine the `final_quality_score`.

### Score thresholds

```yaml
REPORT:
  SCORECARD_OPTIONS:
    max_score: 100
    score_threshold_excellent: 85
    score_threshold_good: 50
    score_threshold_needs_improvement: 0
```

### Final page (optional)

```yaml
REPORT:
  FINAL_PAGE:
    enabled: true
    title: "Report Acknowledgement"
    description: "..."
    link_text: "Call Sights Daily Insights Feedback"
    link_url: "https://..."
```

### TL report options

```yaml
REPORT:
  TL_REPORT_OPTIONS:
    include_yesterday: true
    include_last_week: true
    min_calls_threshold: 1
    excel_format:
      column_width: 25
      font_name: "NotoSansDevanagari"
      font_size: 12
```

### Output directories

```yaml
REPORT:
  OUTPUT:
    pdf_output_dir: "azure://novuscalls/reports/agent_scorecard/2025/10/10/pdf"
    json_output_dir: "azure://novuscalls/reports/agent_scorecard/2025/10/10/json"
    tl_report_output_dir: "azure://novuscalls/reports/tl_reports/2025/10/10/xlsx"
```

### Databricks push

```yaml
REPORT:
  DATABRICKS:
    data_source: "databricks"
    catalog_name: "ildsqlpips"
    schema_name: "retention_intr"
    table_name: "agent_scorecard_table"
    push_to_databricks: true
```

## Date-based path resolution

All report and extraction paths can be auto-resolved by setting `USE_DATE_PATHS: true` in the config or using `--date-path` on the CLI. See `concepts/date-paths.md`.

## Common edits

- Add features: update `features.csv`, run `novus build`, then update `REPORT.FIELD_MAPPING.FEATURES`.
- Change prompt tone: edit `summary/` or `csv_questions/` templates.
- Update output directories: change `OUTPUT` paths or use date resolution.
