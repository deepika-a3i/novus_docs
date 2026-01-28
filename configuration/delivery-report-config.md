# Delivery Report Configuration

The delivery report system generates comprehensive reports on pipeline execution, including run statistics, file-level status, email delivery metrics, and sampling metrics from Databricks.

## Overview

Delivery reports provide:

- **Executive Summary**: Overall run statistics, file processing metrics, and email delivery metrics
- **Sampling Pipeline**: Metrics from the Databricks sampling table
- **Run History**: Detailed list of all pipeline runs with command and subcommand
- **Failed Files**: Files that failed processing with error details
- **Email Details**: Individual email send results with recipient, status, and reason
- **Command Breakdown**: Statistics grouped by command and subcommand

## Configuration File

Create a YAML configuration file for delivery reports:

```yaml
# config/delivery_report_config.yaml

# Email recipients
recipients:
  - email: ops-team@example.com
    name: Operations Team
  - email: manager@example.com
    name: Business Manager

# Sender email address
sender_email: novus-reports@example.com

# Filter by command types (optional - omit to include all)
include_commands:
  - transcribe
  - extract
  - voc-extract
  - report

# Databricks settings (for sampling metrics only)
databricks:
  catalog_name: ildsqlpips
  schema_name: retention_intr
  sampling_table: sampling_metrics

# Path for tracking metadata storage (blob storage)
tracking_path: "azure://novuscalls/run_metadata"

# Failure threshold percentage
failure_threshold_percent: 10.0

# Email subject template
email_subject_template: "Novus Delivery Report - {date_from} to {date_to}"

# Failure notification settings
failure_notification:
  enabled: true
  recipients:
    - ops-alerts@example.com
  sender_email: novus-alerts@example.com
```

## Configuration Fields

### Recipients

| Field | Type | Description |
|-------|------|-------------|
| `email` | string | Recipient email address |
| `name` | string | Recipient display name (optional) |

### Databricks

| Field | Default | Description |
|-------|---------|-------------|
| `catalog_name` | ildsqlpips | Databricks catalog |
| `schema_name` | retention_intr | Databricks schema |
| `sampling_table` | sampling_metrics | Sampling pipeline metrics table |

### Other Settings

| Field | Default | Description |
|-------|---------|-------------|
| `tracking_path` | - | Blob storage path for run metadata |
| `failure_threshold_percent` | 10.0 | Alert threshold for file failures |
| `email_subject_template` | - | Email subject with placeholders |

## CLI Usage

### Generate Report

```bash
# Generate report for yesterday (default)
novus delivery-report --config config/delivery_report_config.yaml

# Specify date range
novus delivery-report --config config/delivery_report_config.yaml \
  --date-from 2025-01-01 --date-to 2025-01-25

# Generate report for a single specific day
novus delivery-report --config config/delivery_report_config.yaml \
  --date-from 2025-01-25

# Save to file
novus delivery-report --config config/delivery_report_config.yaml \
  --output /path/to/report.xlsx

# Generate and send via email
novus delivery-report --config config/delivery_report_config.yaml \
  --send-email

# Full example
novus delivery-report --config config/delivery_report_config.yaml \
  --date-from 2025-01-20 --date-to 2025-01-26 \
  --output reports/delivery_2025-01-26.xlsx \
  --send-email
```

### CLI Options

| Option | Required | Description |
|--------|----------|-------------|
| `--config` | Yes | Path to configuration YAML |
| `--date-from` | No | Start date (YYYY-MM-DD), defaults to yesterday |
| `--date-to` | No | End date (YYYY-MM-DD), defaults to yesterday |
| `--output` | No | Output path for Excel report |
| `--send-email` | No | Send report to configured recipients |
| `--tracking-path` | No | Override tracking metadata path |

### Default Date Behavior

- No dates specified: Reports for **yesterday only**
- Only `--date-from`: Reports from that date **to yesterday**
- Only `--date-to`: Reports for **that single day**
- Both specified: Reports for the **specified range**

## Excel Report Structure

### Sheet 1: Executive Summary

| Metric | Value |
|--------|-------|
| Report Period | 2025-01-25 to 2025-01-25 |
| | |
| Pipeline Runs | |
| Total Runs | 45 |
| Successful Runs | 42 |
| Partial Success | 2 |
| Failed Runs | 0 |
| Aborted Runs | 1 |
| | |
| File Processing | |
| Total Files Processed | 1,250 |
| Successful Files | 1,230 |
| Failed Files | 15 |
| Skipped Files | 5 |
| Success Rate | 98.5% |
| | |
| Email Delivery | |
| Emails Sent | 120 |
| Emails Failed | 3 |
| Emails Skipped | 5 |
| Email Success Rate | 93.8% |

### Sheet 2: Sampling Pipeline

Contains data from the Databricks sampling table:

| agent_id | report_date | total_files | parsed_ok | eligible | sampled | upload_success |
|----------|-------------|-------------|-----------|----------|---------|----------------|
| AGENT001 | 2025-01-25 | 50 | 48 | 45 | 20 | 20 |

### Sheet 3: Run History

| run_id | command | subcommand | started_at | completed_at | status | total_files | success | failed | skipped | emails_sent | emails_failed | emails_skipped | error_message |
|--------|---------|------------|------------|--------------|--------|-------------|---------|--------|---------|-------------|---------------|----------------|---------------|
| abc123 | report | batch | 2025-01-25 10:30 | 2025-01-25 10:45 | success | 150 | 148 | 0 | 2 | 45 | 1 | 2 | |
| def456 | extract | | 2025-01-25 11:00 | 2025-01-25 11:30 | partial_success | 100 | 95 | 5 | 0 | 0 | 0 | 0 | |

### Sheet 4: Failed Files

| run_id | command | file_path | agent_id | error_type | error_message | timestamp |
|--------|---------|-----------|----------|------------|---------------|-----------|
| def456 | extract | trans_015.json | AGENT003 | LLM_TIMEOUT | Request timeout | 2025-01-25 11:15:30 |

### Sheet 5: Email Details

| run_id | command | subcommand | run_date | agent_id | agent_name | email | status | reason |
|--------|---------|------------|----------|----------|------------|-------|--------|--------|
| abc123 | report | batch | 2025-01-25 10:30 | AGENT001 | John Doe | john@example.com | sent | |
| abc123 | report | batch | 2025-01-25 10:30 | AGENT002 | Jane Smith | | skipped | No email address configured |
| abc123 | report | batch | 2025-01-25 10:30 | AGENT003 | Bob Wilson | bob@example.com | failed | SMTP connection error |

### Sheet 6: Command Breakdown

| command | subcommand | total_runs | successful_runs | partial_runs | failed_runs | aborted_runs | files_processed | files_success | files_failed | files_skipped | emails_sent | emails_failed | emails_skipped | file_success_rate | run_success_rate |
|---------|------------|------------|-----------------|--------------|-------------|--------------|-----------------|---------------|--------------|---------------|-------------|---------------|----------------|-------------------|------------------|
| extract | | 15 | 14 | 1 | 0 | 0 | 400 | 395 | 3 | 2 | 0 | 0 | 0 | 98.75 | 93.33 |
| report | batch | 10 | 10 | 0 | 0 | 0 | 300 | 298 | 0 | 2 | 280 | 5 | 15 | 99.33 | 100.0 |
| report | tl-report | 5 | 5 | 0 | 0 | 0 | 25 | 25 | 0 | 0 | 25 | 0 | 0 | 100.0 | 100.0 |
| report | trainer-report | 2 | 2 | 0 | 0 | 0 | 2 | 2 | 0 | 0 | 6 | 0 | 0 | 100.0 | 100.0 |
| transcribe | | 8 | 7 | 0 | 1 | 0 | 200 | 195 | 5 | 0 | 0 | 0 | 0 | 97.5 | 87.5 |
| voc-extract | | 5 | 5 | 0 | 0 | 0 | 150 | 148 | 2 | 0 | 0 | 0 | 0 | 98.67 | 100.0 |

## Email Report

When `--send-email` is used, recipients receive an email with:

**Subject:** `Novus CallSights Delivery Report - 2025-01-25 to 2025-01-25`

**Body:**
```
Novus CallSights Delivery Report
================================

Report Period: 2025-01-25 to 2025-01-25
Generated: 2025-01-26 10:30:45

EXECUTION SUMMARY
-----------------
Total Runs: 15
  - Successful: 12
  - Partial Success: 2
  - Failed: 0
  - Aborted: 1

FILES PROCESSED
---------------
Total Files: 245
  - Successful: 238
  - Failed: 5
  - Skipped: 2
Success Rate: 97.14%

EMAIL DELIVERY
--------------
Emails Sent: 45
Emails Failed: 2
Emails Skipped: 3
Email Success Rate: 90.0%

SAMPLING PIPELINE
-----------------
Total Sampled: 120
Total Uploaded: 118
Upload Errors: 2

---
Please see the attached Excel report for detailed information.

This is an automated report from Novus CallSights.
```

**Attachment:** Excel report with all 6 sheets

## Run Tracking Integration

All CLI commands are automatically instrumented with `RunTracker` when `TRACKING_PATH` is specified in the config. This includes:

- `novus transcribe`
- `novus extract`
- `novus voc-extract`
- `novus report` (single, batch, tl-report, trainer-report)

### Tracked Metrics

**Run-level:**
- Command and subcommand
- Start/end timestamps
- Status (success, partial_success, failed, aborted)
- Total files processed
- Success/failed/skipped counts
- Email metrics (sent/failed/skipped)

**File-level:**
- File path and type
- Processing status
- Error type and message
- Output path

**Email-level:**
- Recipient details (agent_id, agent_name, email)
- Status (sent, failed, skipped)
- Reason for failure/skip

### Enabling Tracking

Add `TRACKING_PATH` to your command config file:

```yaml
TRACKING_PATH: "azure://novuscalls/run_metadata"
```

### Storage Structure

Run metadata is stored at the configured path:

```
<tracking_path>/
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

### Abort Handling

When a command is aborted (Ctrl+C or SIGTERM):
- Tracking data is automatically saved before exit
- Run status is set to `aborted`
- Partial results are preserved

## Failure Notifications

When `failure_notification.enabled` is true, immediate email alerts are sent when:

- A command fails completely (status=failed)
- Critical errors occur (config errors, authentication failures)

These are separate from the scheduled delivery reports and provide real-time alerting.

## Environment Variables

The email service uses these environment variables (see `configuration/environment-variables.md`):

- `EMAIL_TOKEN_URL`
- `EMAIL_API_URL`
- `EMAIL_CLIENT_ID`
- `EMAIL_CLIENT_SECRET`
- `EMAIL_ENCRYPTION_KEY`
- `EMAIL_ALLOWED_DOMAINS`

## Code Location

- CLI Command: `src/novus_calls/cli/commands/delivery_report.py`
- Service: `src/novus_calls/report/delivery_report_service.py`
- Run Tracker: `src/novus_calls/core/tracking/run_tracker.py`
- Tracking Storage: `src/novus_calls/core/tracking/storage.py`
- Tracking Models: `src/novus_calls/core/tracking/models.py`
