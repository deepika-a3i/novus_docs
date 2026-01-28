# Testing and Quality

Testing for Novus CallSights focuses on pipeline smoke tests and schema validation.

## Recommended checks

- Run `novus build` after any CSV or prompt changes.
- Run `novus extract` on a small sample of transcripts.
- Run `novus voc-extract` with `--max-files 3 --no-databricks`.
- Generate one PDF via `novus report single` to verify report templates.

## What to inspect

- Raw JSON outputs for missing or malformed fields.
- Feature-wide tables for column drift or unexpected nulls.
- PDF output formatting and narrative content.

## Optional linting

If `vale` is configured, you can run:

```bash
vale docs/*.md
```

## Performance testing

- Increase `max_concurrent` in VOC configs carefully.
- Use `--max-files` to validate LLM cost behavior before full runs.
