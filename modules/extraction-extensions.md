# Extraction Extensions

Extraction extensions implement individual pipeline steps. They live under `src/novus_calls/core/extraction/extensions/` and are instantiated by the `ExtensionFactory` based on the step `type` in the pipeline config.

## Current extension types

- `plain_text`: ask a free-form question and return raw answer.
- `list_selector`: choose from an enumerated list of options.
- `question`: CSV-driven question step (scorecard features).
- `question_cat_subcat`: question with category/subcategory handling.
- `cat_subcat`: categorical extraction.
- `next_steps`: computes routing based on a prior answer.
- `conditional_branch`: conditional routing helper.
- `split_response`: split multi-part answers.
- `combine_response`: merge response pieces into a single output.
- `combine_headers`: merge or normalize headers for downstream tables.
- `clean_transcript`: normalize transcript text before extraction.
- `keyword_matcher`: rule-based matching against keyword lists.
- `information_value`: extract or score metadata values.
- `no_llm`: placeholder step that does not call an LLM.
- `voc_structured_extraction`: Pydantic-based structured extraction for VOC.
- `voc_list_extraction`: list extraction for VOC detail steps.

## Extension anatomy

Each extension:

- Inherits from the base extension class.
- Implements `run(context, llm)` to produce a structured response.
- Uses prompt templates (`system.md`, `user.md`) and `functions.json` when required.

Extensions are loaded dynamically from disk, so the step `type` must match the extension’s declared `_type`.

## Adding a new extension

1. Create a new extension folder under `src/novus_calls/core/extraction/extensions/`.
2. Implement the class and define a unique `_type`.
3. Register any new config fields you need in the extension’s `to_dict` or config model.
4. Reference the extension `type` in your extraction pipeline YAML.

## Testing tips

- Add a single-step pipeline that only uses the new extension.
- Run `novus extract --config <config>` on a small transcript set.
- Inspect raw JSON outputs for the new fields.
