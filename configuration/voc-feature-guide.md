# Extending VOC Features

The VOC pipeline is fully config-driven. New fields or extraction steps live in YAML, and the CLI reads those files at runtime. Follow this process to add or change Voice of Customer features while keeping the data flow consistent.

## Configuration Map

- `config/voc_config/voc_pipeline.yml`: Declares every pipeline step, routing rule, and output path.
- `config/voc_config/fields/*.yml`: Field schemas (type, description, literals) for structured and list extractions.
- `config/voc_config/prompts/*/{system,user}.md`: Step-specific prompts loaded at runtime.
- `config/voc_config/field_mappings.yml`: Controls which columns flow into the master table and how to merge step outputs.
- `config/voc_config/aggregation_rules.yml`: Aggregates list results and applies custom functions.
- `config/voc_config/final_columns.yml`: Defines the final table schema, types, and Databricks keys.

## Add a Field to an Existing Step

1. Update the relevant YAML file under `config/voc_config/fields/`.

   - Use `type` values supported by the extension (`literal`, `str`, `int`, `float`, `boolean`).
   - For enumerations, list all `options` and craft concise `description` text; the LLM uses it verbatim.
   - Reference shared lists via `dynamic_options` (for example, `${parameters.insurance_providers}`) to keep prompts current.

2. Refresh prompts if context changes.

   - Edit `system.md` or `user.md` inside the matching `prompts/<step>` folder.
   - Keep `{transcript}` and other placeholders untouched.

3. Update table mappings.

   - If the field should ship in the main table, add it to `source_columns` or `merge_configs` in `field_mappings.yml`.
   - Adjust `aggregation_rules.yml` when list outputs require summarisation. Create or reuse custom functions as needed.
   - Append the column name (and data type) to `final_columns.yml`.

4. Validate locally.

   - Run the pipeline on a handful of transcripts:

     ```bash
     novus voc-extract --config config/voc_config/voc_pipeline.yml --max-files 3 --no-databricks
     ```

   - Inspect the raw JSON in your configured `output.raw_features_path` for the new field.
   - Check your configured `output.combined_table_path` to ensure aggregation landed where expected.

## Add a New Conditional Step

1. Author field definitions and prompts in new folders under `fields/` and `prompts/`.
2. Append a step block to `steps` in `voc_pipeline.yml`. Choose `voc_structured_extraction` for single records or `voc_list_extraction` when multiple rows per call are possible.
3. Wire the step into Stage 1 routing by adding an `if_llm_answer` rule under `initial_screening.next_steps`.
4. Extend `aggregation_rules.yml` and `field_mappings.yml` so downstream tables recognise the new step outputs.
5. Update `final_columns.yml` if the new fields must appear in the Delta table or combined Excel.

## Common Checks

- Keep option labels in lowercase; the screening step compares strings exactly.
- Use `max_items` on list steps to prevent runaway token usage.
- When you introduce new custom aggregation functions, document them inside the `implementation` block and keep the code deterministic.
- Always run with `--no-databricks` first; once the schema looks correct, remove the flag to exercise the MERGE logic.
- Commit changes to configs and prompts together. Mismatched field definitions and prompt wording are a common source of extraction drift.
