# Build Pipeline Tutorial

The build step converts an extraction pipeline YAML + CSV feature catalog into a runnable `pipeline.json`. This is required for the agent scorecard extraction flow.

## When you need it

- After editing `features.csv`.
- After editing shared prompt templates under `csv_questions/`.
- When you change any `steps_from_csv` configuration in `extraction_pipeline.yaml`.

If `pipeline.json` is missing or outdated, `novus extract` will refuse to run.

## What build does

`novus build` reads:

- `EXTRACTION.EXTRACTION_PIPELINE_CONFIGURATION` from `pipeline.yaml`.
- `steps_from_csv.source` (the features CSV).
- Shared prompt templates in `steps_from_csv.templates_path`.

It then:

1. Creates a `StepFromCSV` for each row in `features.csv`.
2. Merges those steps with any explicit YAML steps.
3. Loads `system.md`, `user.md`, and `functions.json` for each step.
4. Writes `pipeline.json` next to the extraction YAML.

## Run build

```bash
novus build --config config/agent_scorecard_config/pipeline.yaml
```

Output:

- `config/agent_scorecard_config/pipeline.json`

## Common errors

- **Duplicate step names**: A CSV `abbrev` conflicts with a YAML step name.
- **Missing prompt templates**: `system.md` or `user.md` not found under `csv_questions/`.
- **CSV parsing errors**: unescaped commas or bad quotes in `features.csv`.

## Validation tip

After running build, check `pipeline.json` for:

- Correct `name` per feature.
- Correct function schema parameters.
- Correct model names (inherit from `DEFAULT_LANGUAGE_MODEL` if not specified).
