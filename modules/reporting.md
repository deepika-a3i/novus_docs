# Reporting

Reporting turns the feature-wide table into PDFs, JSON payloads, and email batches for agents, team leads, and trainers.

## Core files

- `src/novus_calls/report/service.py`: main report generation service.
- `src/novus_calls/report/scorecard.py`: scorecard layout and PDF generation.
- `src/novus_calls/report/tl_report_service.py`: TL report generation.
- `src/novus_calls/report/trainer_report_service.py`: trainer/REU reports.

## Responsibilities

- Read feature-wide tables and agent metadata.
- Compute weighted scores and quality bands.
- Generate narrative summaries with LLM prompts.
- Render PDFs and optional JSON payloads.
- Send email batches when configured.

## Configuration inputs

- `REPORT` section in `config/agent_scorecard_config/pipeline.yaml`.
- Email templates: `scorecard_email_config.yaml`, `tl_report_email_config.yaml`, `trainer_report_email_config.yaml`.
- Prompt templates under `summary/` and `csv_questions/`.

## Email config wiring

The report service expects:

```yaml
REPORT:
  EMAIL:
    SCORECARD_CONFIG_PATH: "<config-dir>/scorecard_email_config.yaml"
```

If this block is missing, emails are skipped.

## Output locations

Output directories are defined under:

```yaml
REPORT:
  OUTPUT:
    pdf_output_dir: "..."
    json_output_dir: "..."
    tl_report_output_dir: "..."
```

The CLI will skip generating PDFs if the output file already exists.

## Rendering details

See `modules/report-rendering.md` for how PDFs and Excel reports are rendered.
