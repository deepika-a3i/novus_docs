# Report Customization

The reporting stack turns feature-wide tables into PDFs, JSON payloads, and email batches. Most behavior is controlled by YAML config and prompt templates.

## Key components

- `config/agent_scorecard_config/pipeline.yaml`: report configuration for local runs.
- Production configs should be deployed to your target environment (e.g., `/dbfs/config/` for Databricks).
- `src/novus_calls/report/service.py`: report generation service.
- `src/novus_calls/report/tl_report_service.py`: TL report generation.
- `src/novus_calls/report/trainer_report_service.py`: trainer/REU reports.

## Update narrative tone

- Edit `summary/system.md` for persona shifts or guardrails.
- Edit `summary/user.md` to adjust the framing of the transcript.
- Keep placeholders like `{context}` and `{question}` intact.

## Update feature-level coaching

- Edit `csv_questions/system.md` and `csv_questions/user.md`.
- If you change output schema fields (`answer`, `details`), update `csv_questions/functions.json` and `params_to_generate` in the extraction pipeline config.

## Change scoring weights

In `pipeline.yaml`:

```yaml
REPORT:
  FIELD_MAPPING:
    FEATURES:
      - feature_name: "personalized_pitch"
        weight: 0.20
```

Adjust weights to rebalance the final score. Keep the sum intuitive (often 1.0).

## Add or remove scorecard sections

- Add new features to `REPORT.FIELD_MAPPING.FEATURES` once they exist in extraction outputs.
- Use `display_summary: true` to include in summary sections.

## Configure email behavior

Scorecard emails are driven by a separate YAML file:

```yaml
REPORT:
  EMAIL:
    SCORECARD_CONFIG_PATH: "<config-dir>/scorecard_email_config.yaml"
```

The email config supports subject/body templates, CC/BCC, batching, and attachment rules.

## TL and trainer report email configs

TL and trainer report email configs should be created in your deployment environment with paths like:

- `<config-dir>/tl_report_email_config.yaml`
- `<config-dir>/trainer_report_email_config.yaml`

These configs control recipients and email behavior for higher-level reports.

## Test changes

- Generate a single report before a batch run:

```bash
novus report single \
  --config config/agent_scorecard_config/pipeline.yaml \
  --agent-id AGENT123 \
  --output /tmp/AGENT123.pdf
```

- For TL/trainer reports:

```bash
novus report tl-report --config <config-dir>/tl_report_email_config.yaml
novus report trainer-report --config <config-dir>/trainer_report_email_config.yaml
```
