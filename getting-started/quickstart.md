# Quickstart

This is the shortest path from transcript JSON to a VOC output table.

## 1. Install and activate

```bash
uv sync
source .venv/bin/activate
```

## 2. Configure environment

```bash
cp template.env .env
```

Populate required keys like `OPENAI_API_KEY` and `TEMP_BUCKET`.

## 3. Create a local VOC config

```bash
cp config/voc_config/voc_pipeline.yml config/local_voc.yml
```

Edit `config/local_voc.yml`:

- `input.transcript_path`: `<transcripts-dir>/` (path to your transcript JSON files)
- `output.raw_features_path`: `<output-dir>/voc_features/raw/`
- `output.combined_table_path`: `<output-dir>/voc_features/combined/voc_combined.xlsx`

## 4. Run a small batch

```bash
novus voc-extract --config config/local_voc.yml --max-files 3 --no-databricks
```

## 5. Inspect results

- Raw outputs: `<output-dir>/voc_features/raw/`
- Combined table: `<output-dir>/voc_features/combined/voc_combined.xlsx`

You now have a working local pipeline. Continue with the detailed tutorials for agent scorecards and reporting.
