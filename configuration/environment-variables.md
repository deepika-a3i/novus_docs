# Environment Variables

Copy `template.env` to `.env` and populate variables for your environment. Load them before running any `novus` command.

```bash
cp template.env .env
```

## Core platform

| Variable | Required | Purpose | Notes |
| --- | --- | --- | --- |
| `OPENAI_API_KEY` | Yes | Primary API key used by LiteLLM. | Required even when using Azure OpenAI. |
| `TEMP_BUCKET` | Yes | Scratch location for large intermediate artifacts. | Local path, S3, or Azure URI. |
| `LOG_LEVEL` | No (default `ERROR`) | Logging verbosity for CLI and services. | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`. |
| `LLM_CACHE_DISABLED` | No (default `true`) | Disable LiteLLM disk caching when `true`. | Set to `false` to enable caching. |
| `NOVUS_DATA_PATH` | No | Overrides default local cache folder. | Defaults to `${APPDIR}/novus/data`. |

## LLM routing

| Variable | Required | Purpose | Notes |
| --- | --- | --- | --- |
| `AZURE_OPENAI_ENDPOINT` | When using Azure OpenAI | Base endpoint for Azure-hosted models. | Example: `https://<name>.openai.azure.com/`. |
| `AZURE_OPENAI_API_KEY` | When using Azure OpenAI | Azure OpenAI key. | |
| `AZURE_OPENAI_API_VERSION` | When using Azure OpenAI | API version string passed to LiteLLM. | |
| `OPENAI_API_KEY` | Yes | Shared across OpenAI and Azure routes. | |

## Artifact storage

| Variable | Required | Purpose | Notes |
| --- | --- | --- | --- |
| `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` | When using S3 | Standard AWS credentials. | `AWS_PROFILE` also works. |
| `AWS_DEFAULT_REGION` / `AWS_REGION` | When using S3 | Region fallback. | Defaults to `us-east-1`. |
| `AZURE_STORAGE_CONNECTION_STRING` | Preferred for Azure Blob | Connection string for blob reads/writes. | |
| `AZURE_STORAGE_ACCOUNT_NAME`, `AZURE_STORAGE_ACCOUNT_KEY` | Azure alternative | Account name/key auth. | |
| `AZURE_STORAGE_ACCOUNT_NAME`, `AZURE_STORAGE_SAS_TOKEN` | Azure SAS flow | SAS-authenticated access. | |
| `AZURE_SAS_URL` | Optional | Overrides generated SAS URL. | Rarely needed. |
| `AZURE_STORAGE_CONTAINER_NAME` | Optional | Default container for Azure Speech uploads. | |

## Transcription providers

| Variable | Required | Purpose | Notes |
| --- | --- | --- | --- |
| `AZURE_SPEECH_KEY` | For Azure Speech | Auth key for Azure Speech. | |
| `AZURE_SPEECH_REGION` | For Azure Speech | Region (default `eastus`). | |
| `AZURE_SPEECH_ENDPOINT` | Optional | Custom Azure Speech endpoint. | |
| `DEEPGRAM_API_KEY` | For Deepgram | Deepgram API key. | |

## Email delivery

| Variable | Required | Purpose | Notes |
| --- | --- | --- | --- |
| `EMAIL_TOKEN_URL` | When sending emails | OAuth token endpoint for Graph proxy. | |
| `EMAIL_API_URL` | When sending emails | API endpoint that dispatches mail. | |
| `EMAIL_CLIENT_ID` | When sending emails | OAuth client ID. | |
| `EMAIL_CLIENT_SECRET` | When sending emails | OAuth client secret. | |
| `EMAIL_ENCRYPTION_KEY` | When sending emails | Fernet key for encrypting recipients. | Must be URL-safe base64. |
| `EMAIL_ALLOWED_DOMAINS` | Optional | Comma-separated allowlist for email domains. | Leave blank to allow all. |

## Databricks and Spark

| Variable | Required | Purpose | Notes |
| --- | --- | --- | --- |
| `SPARK_REMOTE` | Optional | Remote Spark Connect URL (`sc://…`). | Used when running off-cluster. |
| `DATABRICKS_SERVER_HOSTNAME` | If using Databricks | Databricks workspace host. | |
| `DATABRICKS_ACCESS_TOKEN` | If using Databricks | Databricks access token. | |
| `DATABRICKS_WAREHOUSE_ID` | If using Databricks SQL connectors | Warehouse ID for SQL operations. | |

## Usage tips

- Load `.env` before running `novus` commands.
- Keep secrets in a vault for production.
- Validate credentials by running a small `--max-files` extraction.
