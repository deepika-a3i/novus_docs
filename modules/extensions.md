# Extensions and Utilities

Extensions are the pluggable units that implement pipeline steps. Utilities provide shared helpers for validation, async handling, and serialization.

## Extension framework

- `src/novus_calls/core/extensions/base.py`: base class for all extensions.
- `src/novus_calls/core/extensions/factory.py`: loads extension classes by name.
- `src/novus_calls/core/extensions/models.py`: extension config models.

## Extraction extensions

Location: `src/novus_calls/core/extraction/extensions/`

Common extensions include:

- VOC structured extraction
- VOC list extraction
- CSV-driven question extraction

These classes take a pipeline step config, build prompts and schemas, and call the LLM via `LiteLLMWrapper`.

## Transcription extensions

Location: `src/novus_calls/core/transcription/extensions/`

- `aws_transcribe.py`
- `azure_speech_transcribe.py`
- `deepgram_transcribe.py`

Each extension adapts a provider API to the standard transcript output format.

## Utilities

Reusable helpers live under `src/novus_calls/core/utils/` and include:

- Async task handling
- Token and cost tracking
- Path normalization
- Serialization and validation
