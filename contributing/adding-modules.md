# Adding Modules or Extensions

This guide covers how to add new pipeline steps or extend existing functionality with new modules.

## Add a new extraction extension

1. Create a new extension class under `src/novus_calls/core/extraction/extensions/`.
2. Implement the `run` method and output model expected by the pipeline.
3. Add any required prompt templates and schema files.
4. Reference the extension by name in the pipeline config `steps` list.

The `ExtensionFactory` loads extensions dynamically based on the `type` in the pipeline config, so naming and module location are important.

## Add a new transcription provider

1. Implement a new provider extension under `src/novus_calls/core/transcription/extensions/`.
2. Update `TranscriptionModelsEnum` and CLI selection logic in `transcribe.py`.
3. Add required environment variables and document them.

## Add a new CLI command

1. Add a new command in `src/novus_calls/cli/commands/`.
2. Register it in `src/novus_calls/cli/__main__.py`.
3. Document usage in `docs/reference/cli-command-reference.md`.

## Add a new report type

1. Create a new report service in `src/novus_calls/report/`.
2. Add a CLI subcommand in `report.py`.
3. Add new templates and config entries in `config/agent_scorecard_config/`.

## Testing checklist

- Run the smallest possible dataset with `--max-files`.
- Validate raw outputs before touching combined tables.
- Update configuration references and docs.
