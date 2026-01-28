# Pipelines

The pipelines module provides the base infrastructure for building step-based processing pipelines. Both the extraction and transcription systems extend this foundation.

## Overview

Pipelines in Novus CallSights are:

- **Step-based**: Composed of named steps that execute in sequence or conditionally
- **Configuration-driven**: Defined via `PipelineConfig` models
- **Protocol-based**: Follow `PipelineProtocol` for consistent interfaces

## Core Classes

### PipelineConfig

Top-level configuration for a pipeline.

```python
from novus_calls.core.pipelines.models import PipelineConfig, PipelineStepConfig

config = PipelineConfig(
    name="voc_extraction",
    initial_step="initial_screening",
    steps=[
        PipelineStepConfig(name="initial_screening", type="voc_structured_extraction"),
        PipelineStepConfig(name="competitor_detail", type="voc_list_extraction"),
    ],
)
```

Fields:

- `name`: Pipeline identifier
- `initial_step`: Name of the first step to execute
- `steps`: List of step configurations

### PipelineStepConfig

Configuration for a single pipeline step.

```python
from novus_calls.core.pipelines.models import PipelineStepConfig

step = PipelineStepConfig(
    name="initial_screening",
    type="voc_structured_extraction",
)
```

Fields:

- `name`: Unique step identifier
- `type`: Extension type that handles this step

### PipelineRunConfig

Configuration passed when running a pipeline.

```python
from novus_calls.core.pipelines.models import PipelineRunConfig

run_config = PipelineRunConfig(
    metadata={"call_id": "CALL001", "agent_id": "AGENT001"},
)
```

### PipelineRunResultConfig

Result returned after pipeline execution.

```python
from novus_calls.core.pipelines.models import PipelineRunResultConfig

# Returned by pipeline.run()
result = PipelineRunResultConfig(
    metadata={"call_id": "CALL001"},
    responses=[step1_result, step2_result],
)
```

Fields:

- `metadata`: Pipeline metadata (call_id, timestamps, etc.)
- `responses`: List of `PipelineStepResult` from each executed step

### PipelineStepResult

Result from a single step execution.

```python
from novus_calls.core.pipelines.models import PipelineStepResult

result = PipelineStepResult(
    extension_name="initial_screening",
)
```

## PipelineBase

Abstract base class implementing `PipelineProtocol`.

```python
from novus_calls.core.pipelines.base import PipelineBase

class MyPipeline(PipelineBase):
    def __init__(self, config: PipelineConfig):
        super().__init__(config)
        self.steps = self._build_steps(config.steps)

    def run(self, run_config: PipelineRunConfig) -> PipelineRunResultConfig:
        # Implementation
        pass
```

Key methods:

- `run(run_config)`: Execute the pipeline (must be implemented by subclasses)
- `find_step_by_name(name)`: Lookup a step configuration by name

## PipelineProtocol

Interface that all pipelines must implement.

```python
from novus_calls.core.pipelines.interfaces import PipelineProtocol

class PipelineProtocol(Protocol[T, R]):
    def run(self, run_config: T) -> R:
        """Execute the pipeline."""
        ...
```

Type parameters:

- `T`: Run configuration type (subclass of `PipelineRunConfig`)
- `R`: Result type (subclass of `PipelineRunResultConfig`)

## Concrete Implementations

### ExtractionPipeline

See `modules/extraction.md` for the extraction pipeline implementation.

```python
from novus_calls.core.extraction.pipeline import ExtractionPipeline
```

### TranscriptionPipeline

See `modules/transcription.md` for the transcription pipeline implementation.

```python
from novus_calls.core.transcription.pipeline import TranscriptionPipeline
```

## Pipeline Flow

```
PipelineConfig
      │
      ▼
  PipelineBase.__init__()
      │
      ▼
  pipeline.run(PipelineRunConfig)
      │
      ▼
  Execute initial_step
      │
      ▼
  Follow next_steps (conditional or sequential)
      │
      ▼
  PipelineRunResultConfig
```

## Code Locations

- `src/novus_calls/core/pipelines/base.py` - PipelineBase class
- `src/novus_calls/core/pipelines/interfaces.py` - PipelineProtocol
- `src/novus_calls/core/pipelines/models.py` - Pydantic models
- `src/novus_calls/core/extraction/pipeline.py` - Extraction implementation
- `src/novus_calls/core/transcription/pipeline.py` - Transcription implementation
