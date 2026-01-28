# Transcription Pipeline Tutorial

This tutorial explains how to run the transcription stage that converts audio to transcript JSON files. The `novus transcribe` command uses a YAML config that maps to `CommandConfig` and its `TRANSCRIPTION` block.

## When to use it

- You have raw audio and need JSON transcripts for extraction.
- You want to test different transcription providers (AWS Transcribe, Azure Speech, Deepgram).
- You need a local cache of transcripts before running extraction or VOC workflows.

## Minimal config

Create a YAML file like `config/local_transcribe.yml`:

```yaml
USER_ID: "local_transcribe"
DESCRIPTION: "Local transcription run"
USECASE: "renewals"
DEFAULT_LANGUAGE_MODEL: "gpt-4.1-mini"

TRANSCRIPTION:
  TRANSCRIPT_MODEL_NAME: "azure_speech"
  AUDIO_PATH: "<audio-dir>/"
  OUTPUT_RAW_PATH: "<transcripts-dir>/raw/"
  OUTPUT_PATH: "<transcripts-dir>/final/"
  OUTPUT_OVERWRITE: false
```

Notes:

- `USECASE` must be `renewals` or `sales`.
- The CLI writes final transcripts under `OUTPUT_PATH/transcription/`.
- If `OUTPUT_OVERWRITE` is `false`, existing transcripts are skipped.

## Run the command

```bash
novus transcribe --config config/local_transcribe.yml
```

## Provider-specific settings

### AWS Transcribe

- Set `TRANSCRIPT_MODEL_NAME: aws_transcribe`.
- Add `TRANSCRIPTION_CONFIGURATION` for mode and AWS settings.
- Provide AWS credentials through environment variables.

### Azure Speech

- Set `TRANSCRIPT_MODEL_NAME: azure_speech`.
- Set `AZURE_SPEECH_KEY` and `AZURE_SPEECH_REGION` in `.env`.

### Deepgram

- Set `TRANSCRIPT_MODEL_NAME: deepgram`.
- Set `DEEPGRAM_API_KEY` in `.env`.

## Advanced flags

### Language identification (LID)

```yaml
TRANSCRIPTION:
  REQUIRE_LID_METADATA: true
  LID_METADATA_PATH: "<transcripts-dir>/lid_metadata.json"
```

### Phrase injection

```yaml
TRANSCRIPTION:
  INJECT_PHRASES: true
  PHRASE_INJECTION_CONFIGURATION:
    injection_phrases: ["policy renewal", "no claim bonus"]
```

### Speaker timestamp removal

```yaml
TRANSCRIPTION:
  SPEAKER_TIMESTAMP_REMOVAL: true
```

## Output layout

- Raw transcripts: `OUTPUT_RAW_PATH` (one JSON per audio file).
- Final transcripts: `OUTPUT_PATH/transcription/`.

These outputs are used by the extraction and VOC commands.
