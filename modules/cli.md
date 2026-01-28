# CLI

The CLI is the main entrypoint for running Novus pipelines. It is implemented with Click and exposes a consistent interface across transcription, extraction, VOC, and reporting workflows.

## Entry point

- Script name: `novus`
- Entry module: `src/novus_calls/cli/__main__.py`

## Command groups

- `novus build`: Generate `pipeline.json` from YAML + CSV definitions.
- `novus transcribe`: Convert audio to transcript JSON.
- `novus extract`: Run the agent scorecard extraction pipeline.
- `novus voc-extract`: Run the VOC pipeline with conditional extraction.
- `novus report`: Generate PDFs and reports (single, batch, tl-report, trainer-report).
- `novus delivery-report`: Generate delivery reports from tracked runs.
- `novus run`: Programmatic wrapper for Databricks notebook integration.

## Date-based path resolution

Several commands support `--target-date` plus `--date-path` / `--no-date-path` flags. When enabled, the CLI resolves transcript, output, and report paths to date-partitioned locations using `DatePathResolver`.

See `concepts/date-paths.md` for details.

## How configs are loaded

- Paths in YAML are normalized by `normalize_config_paths`.
- `CommandConfig` binds to `TRANSCRIPTION`, `EXTRACTION`, and `REPORT` sections.
- The CLI validates required fields before running.

## Code locations

- `src/novus_calls/cli/commands/build.py`
- `src/novus_calls/cli/commands/transcribe.py`
- `src/novus_calls/cli/commands/extract.py`
- `src/novus_calls/cli/commands/voc_extract.py`
- `src/novus_calls/cli/commands/report.py`
- `src/novus_calls/cli/commands/delivery_report.py`
- `src/novus_calls/cli/commands/run.py`

## Programmatic usage (run.py)

The `run.py` module provides `run_with_spark` for invoking CLI commands from Databricks notebooks with Spark session support:

```python
from novus_calls.cli.commands.run import run_with_spark

# Run VOC extraction with Spark session
run_with_spark(
    spark=spark,
    command="voc-extract",
    config="/dbfs/config/voc_pipeline.yml",
    max_files=100,
    databricks=True,
    target_date="2025-10-20",
)

# Run extraction
run_with_spark(
    spark=spark,
    command="extract",
    config="/dbfs/config/pipeline.yaml",
    target_date="2025-10-20",
)

# Generate reports
run_with_spark(
    spark=spark,
    command="report",
    config="/dbfs/config/pipeline.yaml",
    report_type="batch",
    send_email=True,
)
```

A simplified `run_command` interface is also available:

```python
from novus_calls.cli.commands.run import run_command

result = run_command(
    command="build",
    config="config/agent_scorecard_config/pipeline.yaml",
    spark=spark,  # optional
)
```

For a full argument list, see `reference/cli-command-reference.md`.
