# Updating Agent Scorecard Features

Use this guide when you need to add or refine features that feed the agent scorecard. The scorecard pipeline reads configuration from CSV and YAML files, builds a runtime pipeline with `novus build`, and then uses that pipeline during `novus extract` and `novus report`. Small mistakes in the CSV or prompts can block the build step, so work through the checklist in order.

## Key Files

- `config/agent_scorecard_config/features.csv`: Authoritative feature catalogue.
- `config/agent_scorecard_config/csv_questions/`: Shared prompt templates and function schema for CSV-driven steps.
- `config/agent_scorecard_config/pipeline.yaml`: References the CSV, prompt templates, and extraction defaults.
- `config/agent_scorecard_config/extraction_pipeline.yaml`: Extraction flow definition that must stay in sync with `features.csv`.

## Step-by-Step

1. **Draft the new feature row**

   - Copy an existing row in `config/agent_scorecard_config/features.csv`.
   - Update `component_description`, `question`, and `answer_categories`. Keep line breaks inside double quotes.
   - Pick a unique `abbrev` (lowercase snake_case). The build step uses this as the pipeline step name.
   - Set `logic_flow`, `parent`, and `stage_gate` so the conditional routing still works.

2. **Deploy to production**

   - Copy updated configuration to your production environment (e.g., `/dbfs/config/` for Databricks).
   - Ensure the same `features.csv` is used across environments.

3. **Align prompts and function schema**

   - If the shared prompts in `config/agent_scorecard_config/csv_questions/system.md` and `user.md` cover your new feature, no edits are needed.
   - Otherwise, adjust the text or add conditional phrasing. The templates are reused for every CSV-driven feature, so phrase updates carefully.
   - For schema changes (for example, adding a third output field) update `config/agent_scorecard_config/csv_questions/functions.json` and make sure `params_to_generate` in `pipeline.yaml` still matches.

4. **Regenerate the pipeline definition**

   - Run the build command against the config you updated:

     ```bash
     novus build --config config/agent_scorecard_config/pipeline.yaml
     ```

   - The command rewrites `pipeline.json` next to your extraction pipeline YAML.
   - Commit the regenerated JSON so the CLI can load the new step definition.

5. **Smoke-test the pipeline**

   - Execute a small extraction run to prove the new feature populates:

   - ```bash
     novus extract --config config/agent_scorecard_config/pipeline.yaml --target-date YYYY-MM-DD
     ```

   - Inspect the updated call JSON in your configured output directory for the new `<abbrev>_pred` fields.
   - Regenerate a scorecard to confirm the feature influences summaries:

     ```bash
     novus report single --config config/agent_scorecard_config/pipeline.yaml --agent-id AGENT123 --output /tmp/AGENT123.pdf
     ```

6. **Update downstream artefacts if needed**
   - If the new feature should appear in emails or PDFs, edit the relevant templates under `config/agent_scorecard_config/summary/` or the HTML generator in `src/novus_calls/report`.
   - Extend any analytics sheets (for example, feature-wide tables) to include the new columns.

## Tips and Checks

- Keep CSV cells free of stray commas by wrapping multi-line descriptions in quotes.
- Maintain consistent `language_model` values; mixing models within the same batch can slow runs.
- Re-run `novus build` every time you change the CSV or shared prompt templates. The CLI reads pipeline settings from the generated JSON, not directly from the YAML.
- Use a Git diff on `pipeline.json` to verify the new step looks correct (template paths, output names, conditions).
- If the build command fails, check for blank headers or unmatched quotes in the CSV; `pandas.read_csv` is strict and will bubble up parsing errors.
- Document the new feature for coaching teams so they know what behaviour to expect in PDFs.
