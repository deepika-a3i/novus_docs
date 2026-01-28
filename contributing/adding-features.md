# Adding Features

This guide describes how to add new features in both the agent scorecard and VOC pipelines.

## Agent scorecard features

1. Add a new row in `config/agent_scorecard_config/features.csv`.
2. Deploy the updated configuration to your production environment.
3. Update shared prompts or `functions.json` if the schema changes.
4. Run `novus build --config config/agent_scorecard_config/pipeline.yaml`.
5. Run a small extraction and verify the new `<abbrev>_pred` output fields.

Detailed guidance: `configuration/agent-scorecard-features.md`.

## VOC features

1. Update or create a field definition under `config/voc_config/fields/`.
2. Add or adjust prompts in `config/voc_config/prompts/`.
3. Update `field_mappings.yml`, `aggregation_rules.yml`, and `final_columns.yml` as needed.
4. Run `novus voc-extract --config config/voc_config/voc_pipeline.yml --max-files 3 --no-databricks`.

Detailed guidance: `configuration/voc-feature-guide.md`.

## Tips

- Keep feature names consistent across CSV, pipeline JSON, and report config.
- Avoid changing existing output column names without updating downstream reports.
- Version and test prompt changes the same way you test code.
