# Artifact Manager

`ArtifactManager` is the storage abstraction layer used across pipelines. It hides the differences between local paths, S3, and Azure Blob storage so pipeline logic can stay provider-agnostic.

## Responsibilities

- Resolve backend from a URI (`local`, `s3://`, `azure://`).
- Read and write text, bytes, JSON, YAML.
- Read and write DataFrames (CSV and Excel).
- Provide a consistent `exists`, `list_objects`, and `delete` API.

## Key classes

- `ArtifactManager`: `src/novus_calls/core/artifact/artifact_manager.py`
- `ArtifactBackend`: abstract base class defining backend behavior.
- `ArtifactBackendFactory`: creates backend instances based on URI scheme.

## Supported backends

- Local filesystem
- S3 (via boto3)
- Azure Blob (via azure-storage-blob)

## Usage pattern

The CLI and pipeline services typically create a single `ArtifactManager` and pass it into pipeline logic. This keeps reads and writes centralized and lets you swap storage by changing config URIs.

## Common operations

- `read_yaml` to load config files from local or remote locations.
- `list_objects` to discover audio or transcript files.
- `write_df_as_excel` to persist combined tables.

If you add a new storage backend, implement a new `ArtifactBackend` and register it with the factory.
