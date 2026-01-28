# Run Tracking

The tracking module provides automatic run tracking for all CLI commands, capturing execution metadata, file-level processing status, and email delivery results.

## Overview

When `TRACKING_PATH` is configured, all CLI commands automatically track:

- **Run metadata**: Command type, subcommand, timestamps, status, file counts
- **File-level logs**: Individual file processing results with error details
- **Email logs**: Email delivery results with recipient, status, and reason

## Components

### RunTracker

The main class for tracking command execution.

```python
from novus_calls.core.tracking import RunTracker, FileStatus
from novus_calls.core.tracking.storage import TrackingStorage

# Initialize storage
storage = TrackingStorage(
    artifact_manager=artifact_manager,
    base_path="azure://bucket/run_metadata"
)

# Create tracker
tracker = RunTracker(
    command="extract",
    subcommand=None,
    config_path="config/pipeline.yaml",
    storage=storage,
    target_date="2025-01-25",
)

# Start tracking
tracker.start()

# Track individual files
for file in files:
    try:
        process(file)
        tracker.track_file(file, FileStatus.SUCCESS, output_path=output)
    except Exception as e:
        tracker.track_file(
            file,
            FileStatus.FAILED,
            error_type=type(e).__name__,
            error_message=str(e)
        )

# Track email results (for report commands)
tracker.record_email_results({
    "sent": [{"agent_id": "A1", "email": "a@b.com"}],
    "failed": [{"agent_id": "A2", "email": "c@d.com", "error": "SMTP error"}],
    "skipped": [{"agent_id": "A3", "reason": "No email configured"}],
})

# Complete tracking
tracker.complete()
```

### TrackingStorage

Handles persistence of tracking data to blob storage.

```python
from novus_calls.core.tracking.storage import TrackingStorage

storage = TrackingStorage(
    artifact_manager=artifact_manager,
    base_path="azure://bucket/run_metadata"
)

# Save run metadata
storage.save_run_metadata(metadata)

# Save file logs
storage.save_file_logs(file_logs)

# Save email logs
storage.save_email_logs(email_results, run_id)

# List runs for a date range
runs = storage.list_runs(
    start_date=datetime(2025, 1, 20),
    end_date=datetime(2025, 1, 26),
    command="report",  # optional filter
    status=RunStatus.SUCCESS,  # optional filter
)
```

## Data Models

### RunStatus

```python
class RunStatus(str, Enum):
    RUNNING = "running"
    SUCCESS = "success"
    PARTIAL_SUCCESS = "partial_success"
    FAILED = "failed"
    ABORTED = "aborted"
```

### FileStatus

```python
class FileStatus(str, Enum):
    SUCCESS = "success"
    FAILED = "failed"
    SKIPPED = "skipped"
```

### RunMetadata

| Field | Type | Description |
|-------|------|-------------|
| `run_id` | str | Unique identifier (UUID) |
| `command` | str | Command type (transcribe, extract, voc-extract, report) |
| `subcommand` | str | Subcommand (single, batch, tl-report, trainer-report) |
| `config_path` | str | Path to configuration file |
| `target_date` | str | Target date if specified |
| `started_at` | datetime | Run start time |
| `completed_at` | datetime | Run completion time |
| `status` | RunStatus | Final run status |
| `total_files` | int | Total files processed |
| `success_count` | int | Successfully processed files |
| `failed_count` | int | Failed files |
| `skipped_count` | int | Skipped files |
| `emails_sent` | int | Emails successfully sent |
| `emails_failed` | int | Emails that failed |
| `emails_skipped` | int | Emails skipped |
| `email_results` | List[EmailResult] | Detailed email results |
| `error_message` | str | Error message if run failed |

### FileProcessingLog

| Field | Type | Description |
|-------|------|-------------|
| `run_id` | str | Foreign key to run_metadata |
| `file_path` | str | Path to processed file |
| `file_type` | str | Type: audio, transcript, report |
| `agent_id` | str | Agent ID if applicable |
| `status` | FileStatus | Processing status |
| `error_type` | str | Error type if failed |
| `error_message` | str | Error message if failed |
| `processing_time_ms` | int | Processing time |
| `output_path` | str | Output file path |

### EmailResult

| Field | Type | Description |
|-------|------|-------------|
| `agent_id` | str | Agent ID |
| `agent_name` | str | Agent name |
| `email` | str | Email address |
| `status` | str | sent, failed, skipped |
| `reason` | str | Reason for failure/skip |

## Storage Structure

```
<tracking_path>/
  run_metadata/
    YYYY/MM/DD/
      <run_id>.json          # RunMetadata
  file_logs/
    YYYY/MM/DD/
      <run_id>_files.json    # List[FileProcessingLog]
  email_logs/
    YYYY/MM/DD/
      <run_id>_emails.json   # List[EmailResult]
```

## Abort Handling

All CLI commands register signal handlers for graceful abort:

```python
import atexit
import signal

_current_tracker: RunTracker | None = None

def _cleanup_tracker_on_abort(signum=None, frame=None):
    global _current_tracker
    if _current_tracker is not None:
        _current_tracker.complete(
            status=RunStatus.ABORTED,
            error_message=f"Command aborted by user (signal {signum})"
        )
        _current_tracker = None
    if signum is not None:
        raise SystemExit(1)

atexit.register(_cleanup_tracker_on_abort)
signal.signal(signal.SIGINT, _cleanup_tracker_on_abort)
signal.signal(signal.SIGTERM, _cleanup_tracker_on_abort)
```

When a command is aborted (Ctrl+C or SIGTERM):
- Tracking data is saved before exit
- Run status is set to `aborted`
- Partial results are preserved

## Integration with Callbacks

The `TrackerCallbackHandler` bridges the callback system with run tracking, automatically updating the tracker when file events occur:

```python
from novus_calls.core.callbacks import TrackerCallbackHandler, CallbackManager
from novus_calls.core.tracking import RunTracker

tracker = RunTracker(command="extract", config_path="config.yaml")
tracker.start()

# Create handler that bridges callbacks to tracker
handler = TrackerCallbackHandler(tracker)
callback_manager = CallbackManager(handlers=[handler])

# When batch processing fires file events, tracker is automatically updated
batch_manager = callback_manager.on_batch_start(total_files=100)
batch_manager.on_file_end(file_path, 0, 100, "success", duration_ms=1500)
# ^ This automatically calls tracker.track_file()
```

CLI commands automatically set up `TrackerCallbackHandler` when `TRACKING_PATH` is configured. Use `--verbose` to also enable `StdOutCallbackHandler` for debugging.

See `modules/callbacks.md` for more details on the callback system.

## Integration with Delivery Reports

The `novus delivery-report` command reads tracking data to generate comprehensive reports:

```bash
novus delivery-report \
  --config config/delivery_report_config.yaml \
  --date-from 2025-01-20 \
  --date-to 2025-01-26
```

See `configuration/delivery-report-config.md` for details.

## Code Locations

- `src/novus_calls/core/tracking/run_tracker.py` - RunTracker class
- `src/novus_calls/core/tracking/storage.py` - TrackingStorage class
- `src/novus_calls/core/tracking/models.py` - Data models
- `src/novus_calls/core/tracking/__init__.py` - Public exports
