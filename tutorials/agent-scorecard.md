# Agent Scorecard Pipeline Tutorial

This tutorial walks through the agent scorecard pipeline end to end, from transcripts to PDFs and email batches.

## Inputs and configs

Primary config pack: `config/agent_scorecard_config/`

Key files:

- `pipeline.yaml`: main entrypoint used by `novus build`, `novus extract`, and `novus report`.
- `extraction_pipeline.yaml`: extraction flow definition (includes `steps_from_csv`).
- `features.csv`: feature catalog for CSV-driven steps.
- `summary/` and `csv_questions/`: prompt templates for the LLM.

Production configuration should be deployed to your target environment (e.g., `/dbfs/config/` for Databricks).

## Step 1: Build the pipeline JSON

```bash
novus build --config config/agent_scorecard_config/pipeline.yaml
```

This regenerates `pipeline.json` next to `extraction_pipeline.yaml`.

## Step 2: Run extraction

```bash
novus extract \
  --config config/agent_scorecard_config/pipeline.yaml \
  --target-date 2025-10-20 \
  --date-path
```

Outputs:

- Per-call JSON under the configured `OUTPUT_PATH`.
- The feature-wide table: `feature_wide_table.xlsx`.

## Step 3: Generate scorecards

Single agent:

```bash
novus report single \
  --config config/agent_scorecard_config/pipeline.yaml \
  --agent-id AGENT001 \
  --agent-name "Agent 001" \
  --output <reports-dir>/AGENT001.pdf
```

Batch:

```bash
novus report batch --config config/agent_scorecard_config/pipeline.yaml
```

## Step 4: TL and trainer reports

```bash
novus report tl-report --config <config-dir>/tl_report_email_config.yaml
novus report trainer-report --config <config-dir>/trainer_report_email_config.yaml
```

## Email configuration

Scorecard emails are driven by the report config:

```yaml
REPORT:
  EMAIL:
    SCORECARD_CONFIG_PATH: "<config-dir>/scorecard_email_config.yaml"
```

Make sure the referenced YAML exists and has `send_emails: true` if you want emails to send.

## Common adjustments

- Change feature weights in `pipeline.yaml` under `REPORT.FIELD_MAPPING.FEATURES`.
- Update prompt tone in `summary/system.md` or `csv_questions/system.md`.
- Add new features by editing `features.csv` and re-running `novus build`.

For exhaustive configuration details, see `configuration/agent-scorecard-config.md`.
