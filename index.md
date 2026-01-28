# Novus CallSights

Novus CallSights is a modular GenAI platform that turns contact-center audio into structured Voice of Customer insights and coaching-ready scorecards. It is built as a set of pipelines and extensions so you can swap transcription providers, LLM models, and storage backends without rewriting the core workflow.

Use this documentation as a complete guide for:

- Setting up the repository and local environment.
- Understanding the architecture and core modules.
- Running the end-to-end pipelines (transcribe, extract, VOC, report).
- Extending features, prompts, and configurations safely.
- Operating the system in production with Databricks and cloud storage.

## Quick links

- Getting started: `getting-started/setup.md`
- Quickstart walkthrough: `getting-started/quickstart.md`
- Agent scorecard pipeline: `tutorials/agent-scorecard.md`
- VOC pipeline: `tutorials/voc.md`
- CLI reference: `reference/cli-command-reference.md`
- Configuration reference: `configuration/agent-scorecard-config.md`
- Delivery reports: `configuration/delivery-report-config.md`
- Run tracking: `modules/tracking.md`

## What lives where

- `src/novus_calls/`: Core Python package for pipelines, extensions, and services.
- `config/`: Local configuration for VOC and scorecard pipelines.
- `docs/`: MkDocs content generated for this repository.

Data directories (transcripts, outputs, reports) should be configured per deployment via YAML config files.

## Primary workflows

1. Transcribe audio to JSON (`novus transcribe`).
2. Extract structured features from transcripts (`novus extract` or `novus voc-extract`).
3. Combine and score features into tables.
4. Generate reports and PDFs (`novus report`).
5. Monitor pipeline execution with delivery reports (`novus delivery-report`).

Each workflow is fully config-driven and uses the same shared building blocks: storage abstractions, pipeline orchestration, LLM wrappers, and run tracking.
