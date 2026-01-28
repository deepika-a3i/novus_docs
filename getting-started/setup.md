# Setup

This guide walks through setting up the Novus CallSights repo for local development and testing.

## Prerequisites

- Python 3.12
- `uv` for dependency management
- Access to LLM credentials (OpenAI or Azure OpenAI)
- Access to storage credentials if you read or write S3/Azure paths

Optional but common requirements:

- Weasyprint for generating pdf
- System libraries required by `weasyprint` and `pydub`

## Clone and install

```bash
uv sync
source .venv/bin/activate
```

If you use `dotenv`, copy the template and fill the required keys:

```bash
cp template.env .env
```

## Environment variables

The platform reads credentials and settings from environment variables. Start with:

- `OPENAI_API_KEY`
- `TEMP_BUCKET`

Then add provider-specific variables for Azure, AWS, Deepgram, or Databricks as needed.

Full list: `configuration/environment-variables.md`

## Configure local inputs

Create local config files that point to your data directories so you can run smoke tests. Configure the following paths in your YAML configs:

- Transcript paths for input data
- Output paths for extraction results
- Report output directories

Examples:

- VOC pipeline: `config/local_voc.yml` based on `config/voc_config/voc_pipeline.yml`.
- Agent scorecard pipeline: `config/local_scorecard.yaml` based on `config/agent_scorecard_config/pipeline.yaml`.

## Verify install

Run a small VOC extraction on a handful of calls:

```bash
novus voc-extract --config config/local_voc.yml --max-files 3 --no-databricks
```

If this completes, your environment is wired correctly.
