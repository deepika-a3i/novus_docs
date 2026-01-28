# Data Flow and Artifacts

This section describes how data moves through the system and where artifacts are stored.

## Key artifact types

- **Audio files**: Raw `.wav` or `.mp3` inputs for transcription.
- **Transcripts**: JSON files with speaker turns and timestamps.
- **Raw extraction outputs**: Per-step JSON with LLM responses and metadata.
- **Feature-wide tables**: Excel/CSV tables aggregating per-call features.
- **Reports**: PDFs and JSON payloads for agents and team leads.

## Artifact locations

Default paths vary by config. Common output structure:

- `<output-dir>/agent_scorecard/`: Raw extraction outputs and feature tables for scorecards.
- `<output-dir>/voc_features/raw/`: Step-level VOC outputs.
- `<output-dir>/voc_features/combined/`: Combined VOC table.
- `<reports-dir>/`: Generated PDFs.

Configure these paths in your YAML config files.

## Storage backends

`ArtifactManager` abstracts the storage backend. URIs can be:

- Local paths (default)
- `s3://` paths
- `azure://` paths

The same pipeline configs work across backends when the URIs and credentials are configured correctly.

## Pipeline metadata

Each run embeds metadata such as:

- Start time and elapsed time
- Model and token usage
- Input file identifiers

This metadata is stored in raw outputs and can be used for cost tracking and debugging.
