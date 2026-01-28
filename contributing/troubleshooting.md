# Troubleshooting

This section lists common issues and where to look first.

## Build errors

**Symptom:** `novus build` fails with CSV parsing errors.

- Check for unescaped commas or unmatched quotes in `features.csv`.
- Ensure required headers exist and are spelled correctly.

## Missing prompt templates

**Symptom:** extraction fails with "template file not found".

- Verify `templates_path` in the pipeline config.
- Ensure `system.md`/`user.md` (or `system.txt`/`user.txt`) exist.

## LLM auth errors

**Symptom:** LLM requests fail or return 401/403.

- Ensure `OPENAI_API_KEY` or Azure OpenAI variables are set.
- Confirm model name matches configured provider.

## Databricks upsert failures

**Symptom:** MERGE fails or table not found.

- Verify `catalog`, `schema_name`, and `table_name`.
- Check environment variables for Databricks connectivity.
- Run with `--no-databricks` to isolate extraction issues.

## Transcription failures

**Symptom:** No transcripts are created.

- Check `AUDIO_PATH` exists and contains `.wav` or `.mp3` files.
- Confirm provider-specific environment variables.
- Set `OUTPUT_OVERWRITE: true` only if you intend to replace outputs.

## Emails not sending

**Symptom:** `--send-email` logs "SCORECARD_CONFIG_PATH not found" or skips emails.

- Add the email config block under `REPORT.EMAIL.SCORECARD_CONFIG_PATH`.
- Verify the referenced YAML file exists and has `send_emails: true`.
