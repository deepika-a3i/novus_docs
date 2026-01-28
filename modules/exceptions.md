# Exceptions

The `novus_calls.core.exceptions` module provides domain-specific exception classes for Novus CallSights. These replace generic `Exception` usage, improving error handling clarity and enabling more specific error catching.

## Base Exception

### NovusError

Base exception for all Novus CallSights errors.

```python
from novus_calls.core.exceptions import NovusError

raise NovusError("Something went wrong")
```

All other exceptions inherit from `NovusError`, so you can catch all Novus-specific errors with a single handler:

```python
try:
    # ... pipeline code ...
except NovusError as e:
    logger.error(f"Novus error: {e.message}")
```

## Configuration Exceptions

### ConfigurationError

Raised when configuration is invalid or missing.

```python
from novus_calls.core.exceptions import ConfigurationError

raise ConfigurationError("Invalid model name", config_key="LANGUAGE_MODEL_NAME")
```

Attributes:

- `message`: Error description
- `config_key`: Optional key that caused the error

### MissingConfigError

Raised when a required configuration value is missing. Subclass of `ConfigurationError`.

```python
from novus_calls.core.exceptions import MissingConfigError

raise MissingConfigError("OPENAI_API_KEY")
# Message: "Missing required configuration: OPENAI_API_KEY"
```

## Data Exceptions

### DataNotFoundError

Raised when required data is not found.

```python
from novus_calls.core.exceptions import DataNotFoundError

raise DataNotFoundError("transcript_001.json", data_type="Transcript")
# Message: "Transcript not found: transcript_001.json"
```

Attributes:

- `message`: Error description
- `data_type`: Optional type of data that was not found

### DataValidationError

Raised when data validation fails.

```python
from novus_calls.core.exceptions import DataValidationError

raise DataValidationError("must be a positive number", field="score")
# Message: "Validation failed for 'score': must be a positive number"
```

Attributes:

- `message`: Error description
- `field`: Optional field that failed validation

## Report Exceptions

### ReportGenerationError

Raised when report generation fails.

```python
from novus_calls.core.exceptions import ReportGenerationError

raise ReportGenerationError("Failed to render PDF", agent_id="AGENT001")
# Message: "Failed to render PDF (agent: AGENT001)"
```

Attributes:

- `message`: Error description
- `agent_id`: Optional agent ID associated with the failure

### ScorecardError

Raised when scorecard generation fails. Subclass of `ReportGenerationError`.

```python
from novus_calls.core.exceptions import ScorecardError

raise ScorecardError("Missing feature data", agent_id="AGENT001")
```

## Transcription Exceptions

### TranscriptionError

Base exception for transcription operations.

```python
from novus_calls.core.exceptions import TranscriptionError

raise TranscriptionError("Provider timeout", file_path="audio/call_001.wav")
# Message: "Provider timeout (file: audio/call_001.wav)"
```

Attributes:

- `message`: Error description
- `file_path`: Optional path to the audio file

### TranscriptionConfigError

Raised when transcription configuration is invalid. Subclass of `TranscriptionError`.

```python
from novus_calls.core.exceptions import TranscriptionConfigError

raise TranscriptionConfigError("Unknown provider: invalid_provider")
```

### TranscriptionProcessingError

Raised when transcription processing fails. Subclass of `TranscriptionError`.

```python
from novus_calls.core.exceptions import TranscriptionProcessingError

raise TranscriptionProcessingError("Audio format not supported", file_path="call.ogg")
```

## Extraction Exceptions

### ExtractionError

Base exception for extraction operations.

```python
from novus_calls.core.exceptions import ExtractionError

raise ExtractionError("LLM call failed", step_name="initial_screening")
# Message: "LLM call failed (step: initial_screening)"
```

Attributes:

- `message`: Error description
- `step_name`: Optional name of the extraction step

### PipelineExtensionNotFoundError

Raised when a pipeline extension is not found. Subclass of `ExtractionError`.

```python
from novus_calls.core.exceptions import PipelineExtensionNotFoundError

raise PipelineExtensionNotFoundError("custom_extractor")
# Message: "Pipeline extension not found: custom_extractor"
```

### PipelineExtensionRunException

Raised when a pipeline extension fails to run. Subclass of `ExtractionError`.

```python
from novus_calls.core.exceptions import PipelineExtensionRunException

raise PipelineExtensionRunException("Extension returned invalid output", step_name="competitor_detail")
```

## Email Exceptions

### EmailError

Base exception for email operations.

```python
from novus_calls.core.exceptions import EmailError

raise EmailError("SMTP connection failed", recipient="agent@example.com")
# Message: "SMTP connection failed (recipient: agent@example.com)"
```

Attributes:

- `message`: Error description
- `recipient`: Optional email recipient

### EmailAuthenticationError

Raised when email authentication fails. Subclass of `EmailError`.

```python
from novus_calls.core.exceptions import EmailAuthenticationError

raise EmailAuthenticationError("Invalid credentials")
```

### EmailDeliveryError

Raised when email delivery fails. Subclass of `EmailError`.

```python
from novus_calls.core.exceptions import EmailDeliveryError

raise EmailDeliveryError("Mailbox full", recipient="agent@example.com")
```

## Environment Exceptions

### EnvironmentError

Raised when environment configuration is invalid.

```python
from novus_calls.core.exceptions import EnvironmentError

raise EnvironmentError("Invalid format", env_var="TEMP_BUCKET")
# Message: "Invalid format (environment variable: TEMP_BUCKET)"
```

Attributes:

- `message`: Error description
- `env_var`: Optional environment variable name

### MissingEnvironmentVariableError

Raised when a required environment variable is missing. Subclass of `EnvironmentError`.

```python
from novus_calls.core.exceptions import MissingEnvironmentVariableError

raise MissingEnvironmentVariableError("OPENAI_API_KEY")
# Message: "Missing required environment variable: OPENAI_API_KEY"
```

## Exception Hierarchy

```
NovusError
├── ConfigurationError
│   └── MissingConfigError
├── DataNotFoundError
├── DataValidationError
├── ReportGenerationError
│   └── ScorecardError
├── TranscriptionError
│   ├── TranscriptionConfigError
│   └── TranscriptionProcessingError
├── ExtractionError
│   ├── PipelineExtensionNotFoundError
│   └── PipelineExtensionRunException
├── EmailError
│   ├── EmailAuthenticationError
│   └── EmailDeliveryError
└── EnvironmentError
    └── MissingEnvironmentVariableError
```

## Code Location

`src/novus_calls/core/exceptions.py`
