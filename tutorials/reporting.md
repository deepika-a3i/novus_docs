# Reporting and PDF Tutorial

This tutorial covers the reporting stage that turns feature-wide tables into PDFs, JSON payloads, and email batches.

## Inputs

- Feature-wide table (usually `feature_wide_table.xlsx`).
- Report templates under `config/agent_scorecard_config/summary` and `csv_questions`.
- Agent metadata in `config/agent_scorecard_config/agent_details.csv`.
- Optional email config YAMLs in your deployment environment.

## Single report

```bash
novus report single \
  --config config/agent_scorecard_config/pipeline.yaml \
  --agent-id AGENT001 \
  --agent-name "Agent 001" \
  --output <reports-dir>/AGENT001.pdf
```

## Batch report

```bash
novus report batch --config config/agent_scorecard_config/pipeline.yaml
```

## TL and trainer reports

```bash
novus report tl-report --config <config-dir>/tl_report_email_config.yaml
novus report trainer-report --config <config-dir>/trainer_report_email_config.yaml
```

## Date-based paths

```bash
novus report batch \
  --config config/agent_scorecard_config/pipeline.yaml \
  --target-date 2025-10-20 \
  --date-path
```

## Configure scorecard emails

Add the email config to `REPORT`:

```yaml
REPORT:
  EMAIL:
    SCORECARD_CONFIG_PATH: "<config-dir>/scorecard_email_config.yaml"
```

Then run:

```bash
novus report batch --config config/agent_scorecard_config/pipeline.yaml --send-email
```

## Customize report copy

- Edit `summary/system.md` and `summary/user.md` for narrative tone.
- Edit `csv_questions/system.md` and `csv_questions/user.md` for feature-level coaching.
- Keep placeholders like `{context}` and `{question}` unchanged.

Full reference: `configuration/report-customization.md`.
