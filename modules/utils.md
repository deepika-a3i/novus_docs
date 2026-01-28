# Utilities

The utils module provides shared utility functions used across Novus CallSights.

## Environment Tools

### EnvironmentTools

Helper class for reading environment variables with validation.

```python
from novus_calls.core.utils.environment import EnvironmentTools

env = EnvironmentTools()

# Required variable (raises MissingEnvironmentVariableError if not set)
api_key = env.get_required_env_value("OPENAI_API_KEY")

# Optional variable with default
cache_dir = env.get_optional_env_value("LLM_CACHE_DIR", default="/tmp/cache")

# Boolean variable
cache_disabled = env.get_env_bool_value("LLM_CACHE_DISABLED", default=True)
```

Methods:

| Method | Description |
|--------|-------------|
| `get_required_env_value(key)` | Get required variable, raises if missing |
| `get_optional_env_value(key, default)` | Get optional variable with default |
| `get_env_bool_value(key, default)` | Parse boolean from env (true/1/yes/on) |
| `set_central_variables()` | Initialize Config singleton from env |

### Central Variable Initialization

The `set_central_variables()` method initializes the global `Config` singleton:

```python
from novus_calls.core.utils.environment import EnvironmentTools

env = EnvironmentTools()
env.set_central_variables()

# Now Config() has:
# - openai_api_key
# - llm_cache_disabled
# - temp_bucket
# - azure_openai_* (if set)
```

## Email Service

### EmailService

Service for sending emails via Microsoft Graph API with attachment support.

```python
from novus_calls.core.utils.email import EmailService, EmailPayload

service = EmailService()  # Loads config from environment

payload = EmailPayload(
    sender_email="sender@example.com",
    to_email="recipient@example.com",
    subject="Report Ready",
    message="Your scorecard is attached.",
    attachments=["s3://bucket/reports/AGENT001.pdf"],
)

result = service.send_email(payload)
```

### EmailConfig

Configuration loaded from environment variables:

| Variable | Description |
|----------|-------------|
| `EMAIL_TOKEN_URL` | OAuth2 token endpoint |
| `EMAIL_API_URL` | Email API endpoint |
| `EMAIL_CLIENT_ID` | OAuth2 client ID |
| `EMAIL_CLIENT_SECRET` | OAuth2 client secret |
| `EMAIL_ENCRYPTION_KEY` | Fernet key for email encryption |
| `EMAIL_ALLOWED_DOMAINS` | Comma-separated allowed domains |

### Convenience Function

```python
from novus_calls.core.utils.email import send_simple_email

result = send_simple_email(
    sender_email="sender@example.com",
    to_email="recipient@example.com",
    subject="Hello",
    message="This is a test email.",
    attachments=["local/path/file.pdf"],
)
```

### Domain Validation

```python
from novus_calls.core.utils.email import validate_email_domain

# Check if email domain is allowed
is_valid = validate_email_domain("user@company.com", ["company.com"])
# Returns True

# Subdomains are also matched
is_valid = validate_email_domain("user@ext.company.com", ["company.com"])
# Returns True
```

## Async Handler

### AsyncHandler

Utilities for running async code in sync contexts.

```python
from novus_calls.core.utils.async_handler import run_async

# Run async function synchronously
result = run_async(async_function, arg1, arg2)
```

## Timing Utilities

The runtime module provides timing utilities for consistent time handling and performance measurement.

### utcnow

Get timezone-aware UTC timestamp. Use this instead of `datetime.now()` or `datetime.utcnow()`.

```python
from novus_calls import utcnow

# Get current UTC time (timezone-aware)
now = utcnow()
print(now)  # 2025-01-27 14:30:00+00:00
```

### catchtime

Context manager for measuring elapsed time.

```python
from novus_calls import catchtime

with catchtime() as elapsed:
    do_work()

print(f"Took {elapsed():.2f} seconds")
```

### timed_operation

Context manager with optional callback notification.

```python
from novus_calls import timed_operation

# Basic usage
with timed_operation("extract_features") as elapsed:
    do_extraction()
print(f"Took {elapsed():.2f}s")

# With callback notification
with timed_operation("process_file", callbacks=handlers) as elapsed:
    process()
# Callbacks receive on_timing_complete event
```

### @timed Decorator

Decorator for timing functions with optional callback integration.

```python
from novus_calls import timed

# Basic timing (logs to debug)
@timed("process_file")
def process_file(file_path):
    ...

# With callback integration
@timed(callbacks_attr="_callbacks")
def extract(self, data):
    # self._callbacks will be notified of timing
    ...

# Async functions are also supported
@timed("async_operation")
async def fetch_data():
    ...
```

## Logging Configuration

### configure_logging

Set up logging for the application.

```python
from novus_calls.core.utils.log_config import configure_logging

configure_logging()  # Configures root logger with standard format
```

## Validation

### Validation Utilities

Common validation functions.

```python
from novus_calls.core.utils.validation import validate_config

# Validate configuration dictionaries
validate_config(config_dict, required_keys=["key1", "key2"])
```

## Safe Data

### Safe Data Utilities

Utilities for safely handling data transformations.

```python
from novus_calls.core.utils.safe_data import safe_get, safe_parse_json

# Safely get nested dictionary values
value = safe_get(data, "key1.key2.key3", default=None)

# Safely parse JSON with error handling
parsed = safe_parse_json(json_string, default={})
```

## Code Locations

- `src/novus_calls/runtime.py` - Timing utilities (utcnow, catchtime, timed_operation, @timed)
- `src/novus_calls/core/utils/environment.py` - Environment variable handling
- `src/novus_calls/core/utils/email.py` - Email service
- `src/novus_calls/core/utils/async_handler.py` - Async utilities
- `src/novus_calls/core/utils/log_config.py` - Logging configuration
- `src/novus_calls/core/utils/validation.py` - Validation functions
- `src/novus_calls/core/utils/safe_data.py` - Safe data handling
- `src/novus_calls/core/utils/singleton.py` - Singleton pattern utilities
