# Callbacks

The callbacks module provides an event-driven system for monitoring and logging pipeline execution. It enables custom handlers to react to extension, pipeline, batch, and file processing events.

## Overview

Callbacks allow you to:

- Log pipeline execution progress
- Track file processing with automatic RunTracker integration
- Monitor extension start/end/error events
- Handle batch processing events (start, end, per-file)
- Track timing information for operations
- Build custom observability tooling

## Callback Mixins

The callback system is built on composable mixins that define different event types:

### ExtensionManagerMixin

Events for individual extraction extension lifecycle.

```python
def on_extension_start(self, serialized, context, *, name, **kwargs): ...
def on_extension_end(self, outputs, *, name, **kwargs): ...
def on_extension_error(self, error, *, name, **kwargs): ...
```

### PipelineManagerMixin

Events for pipeline-level execution.

```python
def on_pipeline_start(self, pipeline_name, config, *, run_id=None, **kwargs): ...
def on_pipeline_end(self, pipeline_name, results, *, run_id=None, duration_ms=None, **kwargs): ...
def on_step_start(self, step_name, pipeline_name, *, step_index=None, **kwargs): ...
def on_step_end(self, step_name, pipeline_name, outputs, *, duration_ms=None, **kwargs): ...
def on_step_error(self, step_name, pipeline_name, error, **kwargs): ...
```

### BatchManagerMixin

Events for batch file processing.

```python
def on_batch_start(self, total_files, *, batch_id=None, **kwargs): ...
def on_batch_end(self, total_files, success_count, failed_count, *, duration_ms=None, **kwargs): ...
def on_file_start(self, file_path, file_index, total_files, **kwargs): ...
def on_file_end(self, file_path, file_index, total_files, status, *, duration_ms=None, **kwargs): ...
def on_file_error(self, file_path, file_index, total_files, error, **kwargs): ...
```

### TimingManagerMixin

Events for operation timing.

```python
def on_timing_complete(self, operation_name, duration_ms, *, start_time=None, end_time=None, **kwargs): ...
```

## Core Classes

### BaseCallbackHandler

Base class for all callback handlers. Inherits from all mixins.

```python
from novus_calls.core.callbacks import BaseCallbackHandler

class MyHandler(BaseCallbackHandler):
    def on_extension_start(self, serialized, context, *, name, **kwargs):
        print(f"Starting extension: {name}")

    def on_extension_end(self, outputs, *, name, **kwargs):
        print(f"Finished extension: {name}")

    def on_extension_error(self, error, *, name, **kwargs):
        print(f"Error in {name}: {error}")

    def on_file_start(self, file_path, file_index, total_files, **kwargs):
        print(f"Processing file {file_index + 1}/{total_files}: {file_path}")

    def on_file_end(self, file_path, file_index, total_files, status, *, duration_ms=None, **kwargs):
        print(f"File {file_index + 1}/{total_files} completed: {status}")
```

### CallbackManager

Manages a collection of handlers and dispatches events to them.

```python
from novus_calls.core.callbacks import CallbackManager

# Create with handlers
manager = CallbackManager(handlers=[handler1, handler2])

# Add/remove handlers dynamically
manager.add_handler(new_handler)
manager.remove_handler(old_handler)

# Configure from existing callbacks
manager = CallbackManager.configure(
    inheritable_callbacks=[parent_handler],
    local_callbacks=[local_handler],
)
```

### CallbackManagerForExtensionRun

Returned when an extension starts. Used to signal extension end or error.

```python
run_manager = callback_manager.on_extension_start(serialized, context, name="my_step")
run_manager.on_extension_end(outputs, name="my_step")
# Or on error:
run_manager.on_extension_error(exception, name="my_step")
```

### CallbackManagerForPipelineRun

Returned when a pipeline starts. Used for pipeline and step events.

```python
run_manager = callback_manager.on_pipeline_start("extraction", config, run_id="abc123")
run_manager.on_step_start("initial_screening", "extraction", step_index=0)
run_manager.on_step_end("initial_screening", "extraction", outputs, duration_ms=1500)
run_manager.on_pipeline_end("extraction", results, duration_ms=5000)
```

### CallbackManagerForBatchRun

Returned when batch processing starts. Used for file-level events.

```python
run_manager = callback_manager.on_batch_start(total_files=100, batch_id="run-123")
run_manager.on_file_start(file_path, file_index=0, total_files=100)
run_manager.on_file_end(file_path, file_index=0, total_files=100, "success", duration_ms=2000)
run_manager.on_batch_end(total_files=100, success_count=95, failed_count=5, duration_ms=300000)
```

## Built-in Handlers

### StdOutCallbackHandler

Prints all callback events to console with colored output. Useful for debugging.

```python
from novus_calls.core.callbacks import StdOutCallbackHandler

handler = StdOutCallbackHandler()
```

Output example:
```
> Starting pipeline: extraction
  > [0] Step: initial_screening
  > Step initial_screening done (1523ms)
> Pipeline extraction completed (5234ms)

> Processing batch: 100 files
  > [1/100] Processing: transcript_001.json
  > [1/100] SUCCESS (2341ms)
```

### TrackerCallbackHandler

Bridges callbacks with RunTracker for automatic file tracking. This handler automatically updates the RunTracker when file events occur.

```python
from novus_calls.core.callbacks import TrackerCallbackHandler
from novus_calls.core.tracking import RunTracker

tracker = RunTracker(command="extract", config_path="config.yaml")
tracker.start()

handler = TrackerCallbackHandler(tracker)
```

When `on_file_end` or `on_file_error` is called, the handler automatically calls `tracker.track_file()` with the appropriate status and metadata.

### LoggingCallbackHandler

Logs extension events using Python's logging module.

```python
from novus_calls.core.callbacks.logging_handler import LoggingCallbackHandler
import logging

logger = logging.getLogger("pipeline")
handler = LoggingCallbackHandler(logger, log_level=logging.INFO)
```

### FileCallbackHandler

Writes extension events to a file.

```python
from novus_calls.core.callbacks.file_handler import FileCallbackHandler

handler = FileCallbackHandler("pipeline_trace.log", mode="a")
```

## Event Flow

### Extension Events
```
on_extension_start(serialized, context)
         │
         ▼
    Extension runs
         │
         ├─── Success ──► on_extension_end(outputs)
         │
         └─── Failure ──► on_extension_error(error)
```

### Batch Processing Events
```
on_batch_start(total_files)
         │
         ▼
    For each file:
         │
         ├─► on_file_start(file_path, index, total)
         │         │
         │         ▼
         │    Process file
         │         │
         │         ├─── Success ──► on_file_end(..., "success")
         │         │
         │         └─── Failure ──► on_file_error(...)
         │
         ▼
on_batch_end(total, success_count, failed_count)
```

## CLI Integration

All CLI commands support the `--verbose` / `-v` flag to enable `StdOutCallbackHandler` for debugging:

```bash
# Enable verbose callback output
novus extract --config config.yaml --verbose
novus transcribe --config config.yaml -v
novus voc-extract --config config.yaml --verbose
novus report batch --config config.yaml -v
```

Commands automatically set up:
1. `StdOutCallbackHandler` when `--verbose` is enabled
2. `TrackerCallbackHandler` for automatic run tracking (when `TRACKING_PATH` is configured)

## Usage in Extraction Pipelines

Callbacks are passed to extraction pipelines and propagated to extensions:

```python
from novus_calls.core.callbacks import CallbackManager, StdOutCallbackHandler, TrackerCallbackHandler
from novus_calls.core.extraction.pipeline import ExtractionPipeline
from novus_calls.core.tracking import RunTracker

# Set up tracker
tracker = RunTracker(command="extract", config_path="config.yaml")
tracker.start()

# Set up callbacks
callbacks = [
    StdOutCallbackHandler(),
    TrackerCallbackHandler(tracker),
]

# Create pipeline with callbacks
pipeline = ExtractionPipeline(pipeline_config, callbacks=callbacks)

# Run pipeline - callbacks are automatically invoked
result = await pipeline.run(run_config)

# Complete tracking
tracker.complete()
```

## Timing Utilities

The runtime module provides timing utilities that integrate with callbacks:

```python
from novus_calls import utcnow, timed_operation, timed

# Get timezone-aware UTC timestamp
now = utcnow()

# Context manager with callback notification
with timed_operation("extract_features", callbacks=handlers) as elapsed:
    do_extraction()
print(f"Took {elapsed():.2f}s")

# Decorator for timing functions
@timed("process_file")
def process_file(file_path):
    ...

# Decorator with callback integration
@timed(callbacks_attr="_callbacks")
def extract(self, data):
    ...
```

## Async Support

The callback system supports both sync and async handlers. Async coroutines are automatically detected and executed:

- If an event loop is running, callbacks run in a thread pool
- If no event loop exists, a new one is created

## Code Locations

- `src/novus_calls/core/callbacks/base.py` - Base classes and mixins
- `src/novus_calls/core/callbacks/manager.py` - CallbackManager and event handling
- `src/novus_calls/core/callbacks/tracker_handler.py` - TrackerCallbackHandler
- `src/novus_calls/core/callbacks/stdout_handler.py` - Console output handler
- `src/novus_calls/core/callbacks/file_handler.py` - File output handler
- `src/novus_calls/core/callbacks/logging_handler.py` - Logging handler
- `src/novus_calls/core/callbacks/config.py` - Callback configuration utilities
- `src/novus_calls/runtime.py` - Timing utilities (utcnow, timed_operation, @timed)
