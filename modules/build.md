# Build System

The build system converts extraction pipeline YAML and CSV feature definitions into a runtime `pipeline.json`. It is used by the agent scorecard pipeline.

## Why it exists

The scorecard extraction uses CSV-driven features to avoid hand-authoring dozens of pipeline steps. The build process:

- Reads the feature CSV.
- Generates step definitions and function schemas.
- Applies shared prompt templates.
- Writes a fully expanded `pipeline.json` for the extraction engine.

## Core files

- `src/novus_calls/build/build.py`: build orchestration logic.
- `src/novus_calls/build/models.py`: config models for CSV-driven steps.
- `config/agent_scorecard_config/extraction_pipeline.yaml`: describes step wiring and CSV config.

## Key concepts

### `steps_from_csv`

Defines how to turn each CSV row into a pipeline step:

- `source`: path to `features.csv`.
- `templates_path`: prompt templates (`system.md`, `user.md`, `functions.json`).
- `params_to_generate`: which JSON fields should be populated.

### Step merging

The build process combines:

- Explicit YAML steps (summary or screening steps).
- Steps generated from the CSV.

If a CSV `abbrev` matches a YAML step name, build fails to prevent ambiguity.

### Function schema generation

For CSV-driven steps, the build system builds function schemas dynamically from:

- `function_call_*` columns in the CSV.
- `functions.json` template.

This allows consistent structured output without custom code per feature.

## Output

- `pipeline.json` is saved next to the YAML file referenced by `EXTRACTION.EXTRACTION_PIPELINE_CONFIGURATION`.
- `novus extract` uses `pipeline.json` directly.
