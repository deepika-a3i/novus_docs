# Evaluation and Long Table Configuration

The repository includes an `EVALUATION` block in `CommandConfig` for creating long tables and running basic evaluation workflows. These settings live in `ExperimentConfig` (`src/novus_calls/models.py`).

This configuration is optional. Most production flows do not require it, but it is useful for model QA and analysis.

## Where it lives

The `EVALUATION` block sits alongside `TRANSCRIPTION` and `EXTRACTION` in the main config YAML.

Example:

```yaml
EVALUATION:
  AVERAGING_METHOD: "macro"
  GROUND_TRUTH_FILENAME: "<data-dir>/ground_truth.xlsx"
  EVENT_VAL: "yes"
  NON_EVENT_VAL: "no"
  LONG_TABLE_CREATE_CONFIG:
    DATA_FILE: "<output-dir>/agent_scorecard/feature_wide_table.xlsx"
    FEATURE_METADATA_PATH: "config/agent_scorecard_config/features.csv"
    FEATURE_COLUMN: "abbrev"
    FEATURES: ["personalized_pitch", "objection_handling"]
    LONG_TABLE_COLS: ["call_id", "agent_id", "call_date"]
  LONG_TABLE_EVAL_CONFIG:
    EXPERIMENT_LOG_LONG_TABLE: "<output-dir>/experiments/long_table.csv"
    GT_COLUMN: "ground_truth"
    PRED_COLUMN: "prediction"
    EVENT_VAL: "yes"
    F1_AVERAGE: "macro"
    AGGREGATION_KEY: "call_id"
```

## Field reference

### Top-level fields

- `AVERAGING_METHOD`: How to average F1 scores (e.g., `macro`, `micro`).
- `GROUND_TRUTH_FILENAME`: Path to the ground-truth label file.
- `EVENT_VAL` / `NON_EVENT_VAL`: Values used to define positive and negative classes.

### `LONG_TABLE_CREATE_CONFIG`

- `DATA_FILE`: Source data file (feature-wide table).
- `FEATURE_METADATA_PATH`: CSV describing features (often `features.csv`).
- `FEATURE_COLUMN`: Column in the metadata file that holds feature identifiers.
- `FEATURES`: Optional list of features to include.
- `LONG_TABLE_COLS`: Columns to preserve as identifiers.

### `LONG_TABLE_EVAL_CONFIG`

- `EXPERIMENT_LOG_LONG_TABLE`: Output path for evaluation artifacts.
- `GT_COLUMN`: Column containing ground-truth labels.
- `PRED_COLUMN`: Column containing predicted labels.
- `EVENT_VAL`: Value treated as the positive class.
- `F1_AVERAGE`: F1 averaging method.
- `AGGREGATION_KEY`: Identifier to group predictions during evaluation.

## Usage notes

- This configuration is validated but not run by the CLI by default.
- Use it as a structured place to store evaluation metadata and long-table definitions.
