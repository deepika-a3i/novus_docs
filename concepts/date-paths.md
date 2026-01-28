# Date-Based Path Resolution

Many CLI commands can automatically resolve date-partitioned paths for transcripts, outputs, and reports. This behavior is implemented in `DatePathResolver` and is available in `novus extract`, `novus report`, and `novus voc-extract`.

## How it works

- If `USE_DATE_PATHS` is `true` in the config, the CLI will override missing or placeholder paths.
- You can force it on or off using `--date-path` or `--no-date-path`.
- If no target date is supplied, it defaults to yesterday.

## Supported patterns

Resolved paths follow these conventions:

- Transcripts: `transcripts/YYYY/MM/DD/`
- Extraction outputs: `output/<module>/YYYY/MM/DD/`
- Feature wide table: `output/<module>/YYYY/MM/DD/feature_wide_table.xlsx`
- Reports: `reports/<module>/YYYY/MM/DD/<format>/`

The base URI can be `azure://<container>`, `s3://<bucket>`, or a local path.

## CLI flags

- `--target-date YYYY-MM-DD`
- `--date-path` / `--no-date-path`

## Example

```bash
novus extract \
  --config config/agent_scorecard_config/pipeline.yaml \
  --target-date 2025-10-20 \
  --date-path
```

This will resolve transcript, output, and report paths under the date partition for 2025-10-20.
