# CLI Command Reference

The `novus` CLI wraps transcription, extraction, VOC processing, reporting, and pipeline builds. Most commands take a YAML config file and optionally support date-based path resolution.

## Global patterns

- `--config <path>` loads a YAML config file.
- `--target-date YYYY-MM-DD` resolves date-based paths if enabled.
- `--date-path` / `--no-date-path` force path resolution on or off.
- `--verbose` / `-v` enables verbose callback output for debugging (available on most commands).

## Run Tracking

All CLI commands support automatic run tracking when a `TRACKING_PATH` is specified in the config file. This captures:

- **Run metadata**: Command type, subcommand, status, timestamps, file counts, email metrics
- **File-level logs**: Success/failure/skipped status per file with error details
- **Email logs**: Individual email send results with recipient, status, and reason

To enable tracking, add to your config YAML:

```yaml
TRACKING_PATH: "azure://novuscalls/run_metadata"
```

Run metadata is stored at:

```
<TRACKING_PATH>/
  run_metadata/
    YYYY/MM/DD/
      <run_id>.json
  file_logs/
    YYYY/MM/DD/
      <run_id>_files.json
  email_logs/
    YYYY/MM/DD/
      <run_id>_emails.json
```

### Run Status Types

- `success`: All files processed successfully
- `partial_success`: Some files succeeded, some failed
- `failed`: All files failed or critical error occurred
- `aborted`: User-initiated abort (Ctrl+C or SIGTERM)

### Abort Handling

When a command is aborted, tracking data is automatically saved before exit with status `aborted`.

Use `novus delivery-report` to generate reports from tracked runs.

## Failure Notifications

Enable immediate email alerts when a command fails by adding `FAILURE_NOTIFICATION` to your config:

```yaml
FAILURE_NOTIFICATION:
  enabled: true
  recipients:
    - ops-alerts@example.com
    - team-lead@example.com
  sender_email: novus-alerts@example.com
```

When enabled, any command failure will immediately send an alert email containing:

- Command that failed
- Error type and message
- Run summary (files processed, success/failure counts)
- List of failed files with error details

This is separate from the scheduled delivery reports and provides real-time alerting for critical failures.

## novus build

**Purpose:** Generate `pipeline.json` for CSV-driven extraction.

```bash
novus build --config config/agent_scorecard_config/pipeline.yaml
```

Uses:

- `EXTRACTION.EXTRACTION_PIPELINE_CONFIGURATION` from the config.
- `features.csv` referenced in the extraction pipeline YAML.

## novus transcribe

**Purpose:** Convert audio to transcript JSON.

```bash
novus transcribe --config config/transcribe.yaml

# With verbose output for debugging
novus transcribe --config config/transcribe.yaml --verbose
```

Key config fields (`TRANSCRIPTION`):

- `TRANSCRIPT_MODEL_NAME`: `aws_transcribe`, `azure_speech`, or `deepgram`.
- `AUDIO_PATH`: input audio directory.
- `OUTPUT_RAW_PATH`, `OUTPUT_PATH`: where transcripts are written.
- `OUTPUT_OVERWRITE`: skip existing outputs when false.

## novus extract

**Purpose:** Run the agent scorecard extraction pipeline and build a feature-wide table.

```bash
novus extract --config config/agent_scorecard_config/pipeline.yaml --target-date 2025-10-20

# With verbose callback output
novus extract --config config/agent_scorecard_config/pipeline.yaml --target-date 2025-10-20 --verbose
```

Notes:

- Requires `pipeline.json` next to `EXTRACTION.EXTRACTION_PIPELINE_CONFIGURATION`.
- Builds `feature_wide_table.xlsx` in the output path.

## novus voc-extract

**Purpose:** Run the VOC pipeline with conditional extraction and optional Databricks upsert.

```bash
novus voc-extract \
  --config config/voc_config/voc_pipeline.yml \
  --max-files 20 \
  --databricks \
  --target-date 2025-10-20

# With verbose callback output
novus voc-extract --config config/voc_config/voc_pipeline.yml --verbose
```

Flags:

- `--max-files`: limit transcript count.
- `--databricks/--no-databricks`: control Delta MERGE behavior.
- `--date-path`: enable date-based path resolution.
- `--verbose/-v`: enable verbose callback output for debugging.

## novus report

### novus report single

Generate a scorecard for one agent:

```bash
novus report single \
  --config config/agent_scorecard_config/pipeline.yaml \
  --agent-id AGENT001 \
  --agent-name "Agent 001" \
  --output <reports-dir>/AGENT001.pdf

# With verbose callback output
novus report single --config config.yaml --agent-id AGENT001 --verbose
```

Optional:

- `--send-email --email agent@example.com`
- `--verbose/-v`: enable verbose callback output

### novus report batch

Generate reports for all agents (or a subset):

```bash
novus report batch \
  --config config/agent_scorecard_config/pipeline.yaml \
  --agent-ids AGENT001,AGENT002 \
  --send-email

# With verbose callback output
novus report batch --config config.yaml --verbose
```

Optional:

- `--verbose/-v`: enable verbose callback output

### novus report tl-report

Generate daily team-lead reports from Databricks:

```bash
novus report tl-report --config <config-dir>/tl_report_email_config.yaml

# With verbose callback output
novus report tl-report --config <config-dir>/tl_report_email_config.yaml --verbose
```

Optional:

- `--verbose/-v`: enable verbose callback output

### novus report trainer-report

Generate weekly trainer/REU reports:

```bash
novus report trainer-report --config <config-dir>/trainer_report_email_config.yaml

# With verbose callback output
novus report trainer-report --config <config-dir>/trainer_report_email_config.yaml --verbose
```

Optional:

- `--verbose/-v`: enable verbose callback output

## novus delivery-report

**Purpose:** Generate and send delivery reports for pipeline executions.

```bash
# Report for yesterday (default)
novus delivery-report --config config/delivery_report_config.yaml

# Report for specific date range
novus delivery-report \
  --config config/delivery_report_config.yaml \
  --date-from 2025-01-20 \
  --date-to 2025-01-26 \
  --output report.xlsx \
  --send-email

# Report for a single day
novus delivery-report \
  --config config/delivery_report_config.yaml \
  --date-from 2025-01-25
```

Flags:

- `--config`: Path to delivery report configuration YAML (required).
- `--date-from`: Start date (YYYY-MM-DD), defaults to yesterday.
- `--date-to`: End date (YYYY-MM-DD), defaults to yesterday.
- `--output`: Output path for Excel report.
- `--send-email`: Send report to configured recipients.
- `--tracking-path`: Override tracking metadata storage path.

Default date behavior:

- No dates specified: Reports for **yesterday only**
- Only `--date-from`: Reports from that date **to yesterday**
- Only `--date-to`: Reports for **that single day**

Report contents (6 sheets):

- **Executive Summary**: Run statistics, file metrics, email delivery metrics.
- **Sampling Pipeline**: Databricks sampling table metrics.
- **Run History**: Detailed list of pipeline runs with command/subcommand.
- **Failed Files**: Files that failed with error details.
- **Email Details**: Individual email results with recipient, status, reason.
- **Command Breakdown**: Statistics by command and subcommand.

See `configuration/delivery-report-config.md` for configuration details.

## novus run (programmatic helper)

Databricks-friendly wrapper for pipeline commands:

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

Supported commands: `transcribe`, `extract`, `report`, `build`, `voc-extract`.

Report subcommands: `single`, `batch`, `tl-report`, `trainer-report`, `test`.
