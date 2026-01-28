# Extraction Pipeline Configuration

This document describes the extraction pipeline YAML used by the agent scorecard flow (`config/agent_scorecard_config/extraction_pipeline.yaml`). It is the source of truth for how extraction steps are wired, including CSV-driven steps.

## Core structure

An extraction pipeline is defined by:

- `name`: pipeline identifier.
- `initial_step`: name of the first step.
- `steps`: explicit step definitions.
- `steps_from_csv`: rules for generating steps from `features.csv`.

Minimal shape:

```yaml
name: agent_scorecard_pipeline
initial_step: summary
steps:
  - name: summary
    type: plain_text
    templates_path: "config/agent_scorecard_config/summary"
    next_steps:
      - questions
steps_from_csv:
  source: "config/agent_scorecard_config/features.csv"
  templates_path: "config/agent_scorecard_config/csv_questions"
  input_from: summary
  params_to_generate: ["answer", "details"]
```

## Step fields

Each step maps to `ExtractionPipelineStepConfig` in `src/novus_calls/core/extraction/models.py`.

Common fields:

- `name`: unique step name.
- `type`: extension type (mapped to an extension class).
- `templates_path`: folder containing `system.md`, `user.md`, and `functions.json`.
- `question`: optional prompt text appended to templates.
- `input_from`: optional upstream step to pull context from.
- `language_model`: override the model for this step.
- `next_steps`: list of conditional routing rules.
- `transcript_start_fraction` / `transcript_end_fraction`: windowing for long transcripts.

## Conditional routing

`next_steps` supports LLM-driven and metadata-driven conditions.

### LLM answer conditions

```yaml
next_steps:
  - if_llm_answer:
      has_value: "yes"
      go_to: [follow_up_step]
```

To route to CSV-generated steps:

```yaml
next_steps:
  - if_llm_answer:
      has_value: "yes"
      go_to_steps_from_csv_filtered_by:
        parent: level1
        logic_flow: ptp
```

### Metadata conditions

```yaml
next_steps:
  - if_metadata:
      key: call_type
      has_value: intro_call
      go_to: [intro_questions]
```

## CSV-driven steps (`steps_from_csv`)

`steps_from_csv` is used to generate many steps from a feature catalog.

Relevant fields:

- `source`: CSV path (usually `features.csv`).
- `templates_path`: shared prompt templates for all CSV steps.
- `input_from`: step that provides context (often `summary`).
- `params_to_generate`: output fields for the function schema.

Each CSV row becomes a step with:

- `name` from `abbrev`.
- `question` from `question`.
- `language_model` if specified, otherwise `DEFAULT_LANGUAGE_MODEL`.

## Build output

`novus build` compiles this YAML into `pipeline.json`. `novus extract` reads the JSON directly.

## Related docs

- `tutorials/build-pipeline.md`
- `modules/extraction-extensions.md`
