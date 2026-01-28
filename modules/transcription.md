# Transcription Engine

The transcription engine converts audio files into transcript JSON. It supports multiple providers through extension classes, but exposes a single CLI command.

## Core components

- `TranscriptionConfig`: config schema in `src/novus_calls/models.py`.
- `transcribe` command: `src/novus_calls/cli/commands/transcribe.py`.
- Provider extensions:
  - `AWSTranscribeExtension`
  - `AzureSpeechTranscribeExtension`
  - `DeepgramTranscribeExtension`

## Provider selection

The engine selects an extension based on `TRANSCRIPT_MODEL_NAME`:

- `aws_transcribe`
- `azure_speech`
- `deepgram`

## Output behavior

- Raw JSON outputs are written to `OUTPUT_RAW_PATH`.
- Final transcripts are written to `OUTPUT_PATH/transcription/`.
- If `OUTPUT_OVERWRITE` is `false`, the command skips files that already have outputs.

## Related modules

- `src/novus_calls/core/transcription/pipeline.py`
- `src/novus_calls/core/transcription/extensions/`
- `src/novus_calls/core/artifact/` (for file access)
