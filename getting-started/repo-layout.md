# Repository Layout

This repository is split into a Python package and configuration packs. The most important directories are:

- `src/novus_calls/`: Core package with pipelines, extensions, and CLI entrypoints.
- `config/`: Local configuration packs for the agent scorecard and VOC pipelines.
- `docs/`: MkDocs site content.
- `template.env`: Environment variable template.

Data directories (transcripts, outputs, reports) should be configured per deployment via YAML config files. See `configuration/agent-scorecard-config.md` and `configuration/voc-pipeline-config.md` for path configuration.

## Key subfolders

### `src/novus_calls/`

- `cli/`: Click commands for `novus build`, `novus extract`, `novus voc-extract`, `novus report`, `novus transcribe`, `novus delivery-report`, and `novus run`.
- `core/`: Core abstractions (pipelines, extensions, artifact manager, LLM wrapper, transcription, tracking, exceptions).
- `core/tracking/`: Run tracking for monitoring pipeline execution and generating delivery reports.
- `build/`: Pipeline build utilities for generating `pipeline.json` from CSV definitions.
- `report/`: PDF, JSON, email report generation, and delivery report service.
- `external/`: Generated or vendor SDKs (for example, Swagger clients).

### `config/`

- `agent_scorecard_config/`: YAML, CSV, and prompt templates for the scorecard extraction and report pipeline.
- `voc_config/`: YAML, prompt templates, and transformation rules for VOC extraction.

Understanding this layout will make it much easier to navigate the rest of the documentation.
