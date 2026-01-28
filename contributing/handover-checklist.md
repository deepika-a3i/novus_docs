# Handover Checklist

Use this document as a quick-start for new owners of Novus CallSights. It highlights the assets, routines, and decisions that keep the platform healthy in production.

## Repository & Runtime

- **Language:** Python 3.12 (managed via `uv`).
- **Entry point:** `novus` CLI (`novus_calls.cli.__main__`).
- **Key packages:** `litellm`, `delta-spark`, `weasyprint`, `click`, `pandas`.
- **Virtual env:** `uv sync` then `source .venv/bin/activate`.

## Secrets & Configuration

- Populate `.env` using `configuration/environment-variables.md`.
- Store production secrets in a vault (Key Vault, Secrets Manager) and map them to the host environment.
- Keep local `config/` YAML files updated and promote changes through pull requests.

## Data & Storage

- **Transcripts:** Configure via `TRANSCRIPTION.OUTPUT_PATH` in YAML or use remote URIs resolved by `create_resolver_from_config`.
- **Extraction outputs:** Configure via `EXTRACTION.OUTPUT_PATH` in YAML.
- **Reports:** Configure via `REPORT.OUTPUT` in YAML.
- **Databricks tables:** Controlled by the `DATABRICKS` block in each YAML config.

## Operational Runbooks

- **Transcription:** `novus transcribe --config <transcribe.yaml>` converts audio to transcripts.
- **Agent scorecard pipeline:**
  1. `novus build --config config/agent_scorecard_config/pipeline.yaml` (regenerate pipeline).
  2. `novus extract --config … --target-date YYYY-MM-DD`.
  3. `novus report batch --config … --send-email` (optional).
- **VOC pipeline:** `novus voc-extract --config config/voc_config/voc_pipeline.yml --max-files 5 --no-databricks` for smoke tests, remove `--no-databricks` for production runs.
- **Spark/Databricks:** Use `run_with_spark` helper when orchestrating from notebooks.

## Quality & Monitoring

- **Config validation:** Run `novus build` after every CSV or prompt change; review `pipeline.json` diff.
- **Extraction sanity:** Inspect new `<feature>_pred` fields and `feature_wide_table.xlsx` before sending reports.
- **VOC sanity:** Check raw feature JSON and the combined Excel for schema drift.
- **Vale lint:** `vale docs/*.md` before merging documentation changes.
- **Logging:** Controlled by `LOG_LEVEL` and stored in the console or orchestrator logs. Archive run logs with the job metadata.

## Deployment & Release

- Maintain a single main branch; tag releases after passing smoke tests.
- Promote YAML and prompt changes in lockstep with regenerated `pipeline.json`.
- Mirror configs to remote storage such as `/dbfs/config/` before running Databricks jobs.
- Production data should live in managed storage or databases.

## Support & Ownership

- Document escalation paths (LLM provider, cloud storage, email service) in your ops wiki.
- Track outstanding TODOs (future enhancements, confidence scoring) in your issue tracker.
- Keep an eye on API usage in OpenAI/Azure dashboards to avoid throttling.
- Refresh API keys on a schedule and verify access with the small-batch commands listed above.
