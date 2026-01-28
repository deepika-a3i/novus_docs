# System Architecture

This section provides a conceptual overview of how Novus CallSights is structured and how the pieces interact.

## Architectural pillars

1. **Pipeline orchestration**
   - Pipelines are config-driven graphs of steps.
   - Each step is implemented as an extension that can run with or without LLMs.

2. **Storage abstraction**
   - `ArtifactManager` provides a unified interface for local, S3, and Azure Blob storage.
   - Pipelines treat URIs the same regardless of backend.

3. **LLM abstraction**
   - `LiteLLMWrapper` standardizes model calls, structured outputs, and cost tracking.
   - Model configurations are centralized in `LanguageModelConfigs`.

4. **Configuration-driven behavior**
   - YAML, CSV, and prompt templates describe what to extract and how to aggregate.
   - Runtime behavior changes without code changes.

## High-level flow

```
Audio files -> Transcription -> Transcript JSON -> Extraction/VOC -> Combined tables -> Reporting PDFs
```

## Core packages

- `cli/`: Defines commands and marshals configs into runtime services.
- `core/`: Pipeline engine, extensions, artifact management, LLM wrappers.
- `report/`: Reporting services and PDF rendering.

## Extension model

Extensions are pluggable components that implement the logic of a pipeline step. They define:

- How to read inputs from the run context.
- How to call the LLM or other services.
- How to return structured outputs and next steps.

This makes it easy to add new extraction strategies without changing the pipeline runner.
