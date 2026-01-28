# LLM Layer

The LLM layer standardizes how models are called, how structured outputs are parsed, and how costs are tracked.

## Key classes

- `LiteLLMWrapper`: `src/novus_calls/core/llm/main.py`
- `BoundLiteLLM`: helper for structured outputs
- `LanguageModelConfigs`: model config mapping

## Responsibilities

- Normalize call parameters (model, temperature, seed).
- Wrap LiteLLM `acompletion` calls.
- Convert tool-call outputs into JSON.
- Track token usage and cost via `completion_cost`.
- Provide a `with_structured_output` interface similar to LangChain.

## Caching

LiteLLM caching is enabled by default using disk cache unless `llm_cache_disabled` is set. This is useful for reruns of prompt-heavy pipelines.

## Configuration

Models are referenced by name in config files. The model names should exist in `SupportedLanguageModels` or be available via LiteLLM.

When using Azure OpenAI, provide:

- `AZURE_OPENAI_ENDPOINT`
- `AZURE_OPENAI_API_KEY`
- `AZURE_OPENAI_API_VERSION`

For OpenAI:

- `OPENAI_API_KEY`
