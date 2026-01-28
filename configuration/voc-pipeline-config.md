# VOC Pipeline Configuration Reference

This document describes the configuration for the **Voice of Customer (VOC)** pipeline, based on the files under:

- `config/voc_config/`

It is intended as a self‑contained handover reference for:

- Understanding what each configuration field does.
- Knowing the effect of changing individual fields.
- Extending the VOC pipeline with new steps, fields, and outputs.

All descriptions below are derived only from `config/voc_config/voc_pipeline.yml` and the other files in the same folder.

For high‑level conceptual overview of the VOC system, see `configuration/voc-description.md`. For how to extend features in practice, see `configuration/voc-feature-guide.md`. This file focuses on the **config schema and behavior**.

---

## 1. Folder layout and roles

Root: `config/voc_config/`

- `voc_pipeline.yml`
  Main pipeline configuration:
  - Orchestrates the full extraction flow.
  - Defines input/output locations, models, and conditional logic.
  - Wires transformations and Databricks export.

- `fields/*.yml`
  Field definitions for each extraction step (both structured and list):
  - Names, types, descriptions, enum options.
  - Required flags and dynamic option sources.

- `prompts/*/`
  Prompt templates per step:
  - `system.txt`
  - `user.txt` (for steps that need a user template)

- `parameters/`
  Shared parameter lists used across steps:
  - `insurance_providers.txt`
  - `coverage_gaps.yml`

- `field_mappings.yml`
  Mapping and merge configuration:
  - Which columns to copy from raw outputs.
  - How to derive new columns.
  - How to merge detailed tables into a single combined table.

- `aggregation_rules.yml`
  Aggregation rules and custom functions:
  - How to reduce list outputs (e.g. competitor list) to single‑row features.
  - Python‑style helper functions embedded as strings.

- `final_columns.yml`
  Final combined table schema:
  - Column order.
  - Data types and documentation.
  - Primary key and partitioning for Databricks.

Production configuration should be deployed to your target environment (e.g., `/dbfs/config/` for Databricks). This document treats `config/voc_config/` as the canonical configuration.

---

## 2. `voc_pipeline.yml` – top‑level pipeline configuration

Path:
`config/voc_config/voc_pipeline.yml`

### 2.1 Top‑level metadata

```yaml
name: voice_of_customer_pipeline
initial_step: initial_screening
max_concurrent: 5
```

| Field            | Type | Description & effect                                                                                                                                                               |
| ---------------- | ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`           | str  | Logical name of the pipeline. Used for logging/monitoring and high‑level identification. Changing this does not affect behavior unless referenced elsewhere.                       |
| `initial_step`   | str  | Name of the first step to run. Must match one of the entries in `steps`. Changing this changes where the pipeline starts (e.g. if you ever introduce a pre‑screening step).        |
| `max_concurrent` | int  | Maximum number of calls (or transcripts) processed in parallel. Increasing this can improve throughput but may increase resource usage or rate‑limit pressure on the LLM provider. |

---

### 2.2 `base_config_folder`

```yaml
base_config_folder: "/Volumes/ildsqlpips/retention_intr/retention_files/Callsights/artifacts/config/voc/"
```

`base_config_folder` is the **root for all relative config paths** (fields, prompts, parameters, transformation files). When the engine resolves a path like `fields/initial_screening_fields.yml`, it effectively uses:

`<base_config_folder>/fields/initial_screening_fields.yml`

Changing this:

- Lets you move the entire VOC config pack to a different location.
- Requires the same directory structure (`fields`, `prompts`, `parameters`, etc.) to exist under the new base folder.

For deployment, set `base_config_folder` to your target configuration directory (e.g., `/dbfs/config/voc/` for Databricks).

---

### 2.3 Input and output configuration

```yaml
input:
  transcript_path: "azure://novuscalls/transcripts/2025/10/10"

output:
  raw_features_path: "azure://novuscalls/output/voc/2025/10/10/"
  combined_table_path: "azure://novuscalls/output/voc/2025/10/10/voc_combined.xlsx"
```

**Input**

| Field                   | Type     | Description & effect                                                                                                                                                                                                                     |
| ----------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `input.transcript_path` | str/path | Source location of call transcripts. Usually a date‑partitioned Azure container path. Changing this tells the pipeline which calls to process for a given run. Make sure the transcript format matches what the extraction code expects. |

**Output**

| Field                        | Type     | Description & effect                                                                                                                                                                                                                   |
| ---------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `output.raw_features_path`   | str/path | Base folder for raw per‑step extraction outputs (e.g. JSON or parquet produced by each step). Changing this moves where raw intermediate artifacts are stored.                                                                         |
| `output.combined_table_path` | str/path | Path to the final combined VOC table (e.g. Excel). This table is built using `field_mappings.yml`, `aggregation_rules.yml`, and `final_columns.yml`. Changing it moves the location of this combined output only; logic is unaffected. |

**Typical change pattern**

For a new processing date:

- Update `input.transcript_path`.
- Update `output.raw_features_path` and `output.combined_table_path` to the matching date slice.

---

### 2.4 `parameters` – external reference files

```yaml
parameters:
  insurance_providers: "parameters/insurance_providers.txt"
  coverage_gaps: "parameters/coverage_gaps.yml"
```

These are shared lists or dictionaries used in multiple steps.

| Field                            | Type | Description & effect                                                                                                                                                                                        |
| -------------------------------- | ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `parameters.insurance_providers` | path | Text file containing names of insurance providers. Used, for example, in competitor‑related prompts to help the LLM recognize and normalize competitor names. Passed into steps via the `parameters` block. |
| `parameters.coverage_gaps`       | path | YAML configuration describing coverage gap categories. Used in the coverage gap detail step to standardize gap names and descriptions.                                                                      |

Paths here are **relative to `base_config_folder`**. Changing either path:

- Swaps in a different reference list.
- Requires the new file to exist and follow the expected format (plain text list vs YAML).

---

## 3. Pipeline steps and branching

### 3.1 Step overview

The `steps` array defines the entire execution graph:

```yaml
steps:
  - name: initial_screening
    type: voc_structured_extraction
    language_model: "gpt-5-mini"
    fields_config: "fields/initial_screening_fields.yml"
    templates_path: "prompts/initial_screening"
    parameters:
      insurance_providers: "${parameters.insurance_providers}"
    next_steps:
      ...

  - name: will_renew_later_detail
    type: voc_structured_extraction
    ...

  - name: competitor_detail
    type: voc_list_extraction
    ...
```

There are two main categories of steps:

- **Structured extraction (`voc_structured_extraction`)** – returns exactly one record per call, with fields defined in a `fields/*.yml` file.
- **List extraction (`voc_list_extraction`)** – returns zero or more records per call (lists of issues or competitors), later aggregated by `aggregation_rules.yml`.

All steps share a common pattern of fields:

| Field            | Description                                                                                                                                         |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`           | Unique identifier of the step. Used for routing (`go_to`) and in downstream transformation configs (`field_mappings.yml`, `aggregation_rules.yml`). |
| `type`           | Either `voc_structured_extraction` or `voc_list_extraction`. Determines the expected output shape and which engine is used.                         |
| `language_model` | LLM model name used for this step (here: `"gpt-5-mini"` for all steps). You can change this per step if needed.                                     |
| `fields_config`  | Relative path (from `base_config_folder`) to the field definition YAML for this step. Changing this swaps in a different schema for that step.      |
| `templates_path` | Folder containing `system.txt` and optionally `user.txt` prompts for the step. Changing this changes how the prompt is phrased.                     |
| `parameters`     | Optional key‑value pairs injecting shared parameter files into the step. Values can refer to `parameters.*` via `${parameters.<name>}`.             |
| `max_items`      | (List steps only) Upper bound on the number of list items to extract per call. Controls token usage and output size.                                |
| `next_steps`     | Optional list of conditional routing rules evaluated after the step finishes.                                                                       |

---

### 3.2 Stage 1 – `initial_screening`

The first step:

```yaml
- name: initial_screening
  type: voc_structured_extraction
  language_model: "gpt-5-mini"
  fields_config: "fields/initial_screening_fields.yml"
  templates_path: "prompts/initial_screening"
  parameters:
    insurance_providers: "${parameters.insurance_providers}"
  next_steps:
    ...
```

**Behavior:**

- Reads the transcript and extracts **high‑level flags and preferences** defined in `fields/initial_screening_fields.yml`:
  - Renewal intent.
  - Yes/no flags for different issue categories.
  - Customer sentiment.
  - Preferred contact channel and language.
- Uses prompts from `prompts/initial_screening`.
- Injects the `insurance_providers` list into the prompt (so the LLM can better interpret competitor mentions).

**Routing logic (`next_steps`):**

`next_steps` uses a set of `if_llm_answer` rules to **branch into Stage 2 steps** based on the structured outputs from `initial_screening`:

```yaml
next_steps:
  # Renewal intent branches
  - if_llm_answer:
      field: "renewal_intent"
      has_value: "will_renew_but_later"
      go_to: [will_renew_later_detail]
  ...
  # Issue-based branches
  - if_llm_answer:
      field: "competitor_mentioned"
      has_value: "yes"
      go_to: [competitor_detail]
  ...
```

Each rule means:

- Look at the structured output field `field`.
- If its value equals `has_value`, enqueue the listed `go_to` steps for this call.

**Important:**

- Field names used here (e.g. `renewal_intent`, `competitor_mentioned`, `pricing_issue`) must match field names defined in `fields/initial_screening_fields.yml`.
- Allowed values (e.g. `"yes"`, `"no"`, `"will_not_renew"`) must match the enumerations and casing used there.

Changing a routing rule:

- Adjust the `field` name if the schema changes.
- Adjust `has_value` to match new categories.
- Add or remove `go_to` steps to change which detailed extractions run.

---

### 3.3 Stage 2 – detailed extraction steps

Stage 2 is built entirely from `voc_structured_extraction` and `voc_list_extraction` steps that run only when triggered by Stage 1 flags.

#### 3.3.1 Renewal intent details (single record per call)

Steps:

- `will_renew_later_detail`
- `unable_to_renew_detail`
- `will_not_renew_detail`
- `undecided_detail`

Each uses:

```yaml
- name: will_renew_later_detail
  type: voc_structured_extraction
  language_model: "gpt-5-mini"
  fields_config: "fields/will_renew_later_fields.yml"
  templates_path: "prompts/will_renew_later"
  next_steps: []
```

Semantics:

- Exactly one of these steps will run for a call, based on `initial_screening.renewal_intent`.
- Each collects:
  - A **reason** (from a controlled set of literals).
  - Additional free‑text comments and, where relevant, a timeline.
- Outputs are merged back into the main table via `field_mappings.yml` (see Section 4.1).

You can adjust:

- The detailed schema in `fields/*_fields.yml`.
- The narrative prompts in `prompts/*`.

#### 3.3.2 Issue details (possibly multiple items per call)

Steps:

- `competitor_detail` – list of competitor mentions.
- `pricing_issue_detail` – structured pricing complaints.
- `claims_issue_detail` – structured claims issues.
- `payment_issue_detail` – structured payment problems.
- `communication_issue_detail` – structured communication issues.
- `coverage_gap_detail` – list of coverage gaps.
- `service_issue_detail` – structured service issues.

Example (`competitor_detail`):

```yaml
- name: competitor_detail
  type: voc_list_extraction
  language_model: "gpt-5-mini"
  fields_config: "fields/competitor_fields.yml"
  templates_path: "prompts/competitor"
  parameters:
    insurance_providers: "${parameters.insurance_providers}"
  max_items: 10
  next_steps: []
```

Key behaviors:

- `voc_list_extraction` can return multiple rows per call (e.g. several competitors, several issues).
- `max_items` bounds the number of rows, controlling cost.
- Schema and options are defined in the corresponding `fields/*.yml` file.
- Aggregation into single‑row features is handled later via `aggregation_rules.yml`.

Changing these steps:

- Update `fields_config` content to change what is captured.
- Update `templates_path` prompts to modify instructions without changing code.
- Tune `max_items` based on typical number of events per call.

---

## 4. Transformation and table building

After all extraction steps complete, the pipeline combines outputs into a **single per‑call table** using the `transformation` section and its referenced files.

### 4.1 `transformation` block in `voc_pipeline.yml`

```yaml
transformation:
  field_mappings: "field_mappings.yml"
  aggregation_rules: "aggregation_rules.yml"
  final_columns: "final_columns.yml"
```

Each path is relative to `base_config_folder`:

| Field               | Description & effect                                                                                                  |
| ------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `field_mappings`    | Points to `field_mappings.yml`, which defines how raw outputs map into a wide table and how detail tables are merged. |
| `aggregation_rules` | Points to `aggregation_rules.yml`, which reduces list tables (e.g. competitors) into aggregated fields.               |
| `final_columns`     | Points to `final_columns.yml`, which specifies the schema, types, and column order for the final combined table.      |

Changing any of these paths swaps in a different transformation spec; normally you only change the **content** of these files.

---

### 4.2 `field_mappings.yml` – mapping and merging

Path:
`config/voc_config/field_mappings.yml`

Key sections:

#### 4.2.1 `full_table.source_columns`

```yaml
full_table:
  source_columns:
    - filename
    - call_uuid
    - call_date
    ...
    - response_preferred_language
```

Defines which columns to copy from the **initial screening** output into the base table.

- Source columns must exist in the structured output of `initial_screening`.
- To expose a new flag or field from screening in the final table, add it here (and later to `final_columns.yml`).

#### 4.2.2 `full_table.column_renames`

```yaml
column_renames:
  response_renewal_intent: renewal_intent
  response_pricing_issue: pricing_issue_mentioned
  ...
```

Renames verbose internal column names (e.g. `response_*`) to cleaner, business‑friendly names used in the final table.

- Left side: original column from screening output.
- Right side: new column name used in downstream configs (`final_columns.yml`, Databricks).

Changing names here requires keeping other configs (e.g. `final_columns.yml`) in sync.

#### 4.2.3 `full_table.derived_columns`

```yaml
derived_columns:
  - name: mobilenum
    type: function
    function: extract_mobile_number
    args: [call_id]
    description: "Extract 10-digit mobile number from call_id"
  - name: call_year
    type: datetime_component
    source: call_date
    component: year
  ...
```

Defines columns computed from other fields:

- `type: function` – calls a named custom function defined in `aggregation_rules.yml`’s `custom_functions` block (e.g. `extract_mobile_number`).
- `type: datetime_component` – extracts components (year, month, day, hour) from a date/time column.

To add a new derived column:

1. Add a new entry here.
2. If `type: function`, define the function in `aggregation_rules.yml`.
3. Add the column and type in `final_columns.yml`.

#### 4.2.4 `merge_configs`

```yaml
merge_configs:
  - table: will_renew_later_detail
    merge_columns:
      - filename
    select_columns:
      - filename
      - reason
      - comments
    rename_columns:
      reason: renewal_intent_reason
      comments: renewal_intent_comments
  ...
```

Defines how to join **Stage 2 detail tables** back into the main table:

- `table` – name of the detail table, typically equal to the step name in `voc_pipeline.yml`.
- `merge_columns` – join keys (usually `filename` linking transcripts).
- `select_columns` – which columns to carry over from the detail table.
- `rename_columns` – how to rename detail fields into their final names (e.g. mapping many detail steps into shared columns like `renewal_intent_reason`).

Effect:

- Different renewal intent detail steps (will renew later, unable to renew, etc.) all map their `reason` and `comments` into a unified `renewal_intent_reason` and `renewal_intent_comments` set of columns.

#### 4.2.5 `default_values`

```yaml
default_values:
  renewal_intent_reason: "renewed"
```

Specifies default values when merged columns are missing (e.g. when no detail step ran). In this case, if no detail reason is found, `renewal_intent_reason` falls back to `"renewed"`.

---

### 4.3 `aggregation_rules.yml` – list aggregation and custom functions

Path:
`config/voc_config/aggregation_rules.yml`

Structure:

- `aggregations` – rules for aggregating multi‑row tables (e.g. competitor, pricing, claims, coverage, service).
- `custom_functions` – Python‑style helper functions used in aggregation and derived columns.

#### 4.3.1 `aggregations`

Example (competitor aggregation):

```yaml
aggregations:
  - table: competitor_detail
    group_by: filename
    rules:
      name:
        method: join_unique
        separator: "|"
        output_name: competitor_names
      premium:
        method: custom
        function: find_competitor_price
        output_name: competitor_premium
      ...
```

Key concepts:

- `table` – name of a list step’s output table (e.g. `competitor_detail`).
- `group_by` – columns to group by when aggregating (typically `filename` = one row per call).
- `rules` – per‑column aggregation rules:
  - `method: join_unique` – join distinct values with `separator`.
  - `method: custom` – call a named custom function; expects `function` and `output_name`.
  - `na_filter: true` – drop null/NA values before aggregation.

Other sections (pricing issues, claims, coverage gaps, etc.) follow this pattern, generating fields such as:

- `pricing_issue_type`, `premium_on_call`, `insured_value_on_call`, `pricing_issue_comments`.
- `claims_issue_type`, `claim_amount`, `claims_disbursed_amount`, `claims_comments`, `claims_reimbursement_status`.
- `coverage_gap_name`, `coverage_gap_comments`.
- `service_issue_type`, `service_issue_comments`.

#### 4.3.2 `custom_functions`

Example functions:

- `find_price` – cleans and combines price values, enforcing sanity checks (e.g. upper bound on price).
- `find_competitor_price` – filters invalid competitor price values and joins them into a pipe‑separated string.
- `extract_mobile_number` – extracts a 10‑digit mobile number from various call ID formats.
- `sum_normalized` – sums numeric values and normalizes by count, returning an integer.

These functions are embedded as code strings and executed inside the transformation logic.

When adding new aggregation behavior:

1. Define a new function under `custom_functions`.
2. Reference it from an aggregation rule or `derived_columns` entry via `function: <name>`.

---

### 4.4 `final_columns.yml` – final schema and Databricks configuration

Path:
`config/voc_config/final_columns.yml`

Key sections:

#### 4.4.1 `columns`

```yaml
columns:
  - mobilenum
  - filename
  - call_uuid
  - call_date
  ...
  - preferred_language
```

Defines:

- Exact column order for the final combined VOC table.
- Must include all columns produced by:
  - `field_mappings.yml` (source + renamed + derived columns).
  - `aggregation_rules.yml` (aggregation outputs).

To add a new column to the final table:

1. Ensure it is produced by a mapping, merge, or aggregation.
2. Append it here at the appropriate position.
3. Update `data_types` and optionally `column_descriptions`.

#### 4.4.2 `primary_key` and `partition_by`

```yaml
primary_key:
  - filename

partition_by:
  - call_year
  - call_month
```

Used by the Databricks upsert logic:

- `primary_key` – logical primary key for MERGE operations (can be composite if needed).
- `partition_by` – partitions for the Delta table (here by call year and month).

Changing these requires:

- Matching changes in Databricks table definition and queries.
- Ensuring the columns exist and are populated in the combined table.

#### 4.4.3 `data_types`

Defines the expected data type for each column:

- `string`, `integer`, `float`, `date`.

These are used for:

- Validation.
- Casting when writing out the final table or pushing to Databricks.

If you add a new column, you should:

- Add its name and type to `data_types`.
- Ensure upstream data is compatible with that type.

#### 4.4.4 `column_descriptions`

Provides human‑readable descriptions for key columns, e.g.:

- `mobilenum`: 10‑digit mobile number extracted from `call_id`.
- `renewal_intent`: customer’s renewal intention category.
- `customer_sentiment`: overall sentiment (positive/neutral/negative).

These are primarily for documentation and tooling (e.g. data catalogs).

---

## 5. Databricks integration

From `voc_pipeline.yml`:

```yaml
databricks:
  catalog: "ildsqlpips"
  schema_name: "retention_intr"
  table_name: "temp_voc_metadata"
  merge_keys: ["filename"]
  partition_by: ["call_year", "call_month"]
  enable_upsert: true
```

| Field           | Type | Description & effect                                                                                                       |
| --------------- | ---- | -------------------------------------------------------------------------------------------------------------------------- |
| `catalog`       | str  | Databricks catalog name containing the target schema.                                                                      |
| `schema_name`   | str  | Databricks schema (database) where the table lives.                                                                        |
| `table_name`    | str  | Name of the target Delta table.                                                                                            |
| `merge_keys`    | list | List of columns used as MERGE keys (should align with `primary_key` in `final_columns.yml`).                               |
| `partition_by`  | list | Columns used to partition the Delta table; should align with `partition_by` in `final_columns.yml`.                        |
| `enable_upsert` | bool | If `true`, perform MERGE/UPSERT into the table. If `false`, skip Databricks writes (useful for dry runs or local testing). |

When changing:

- To **disable** Databricks writes: set `enable_upsert: false`.
- To point to a new table or database:
  - Update `catalog`, `schema_name`, and `table_name`.
  - Ensure the target table schema matches the final columns schema (or is compatible).

---

## 6. Metadata block

```yaml
metadata:
  version: "1.0"
  created: "2025-10-08"
  description: "Voice of Customer analysis pipeline for ICICI Lombard renewal calls"
  stages:
    - name: "initial_screening"
      description: "Extract 9 high-level flags using gpt-5-mini for cost efficiency"
    - name: "detailed_extraction"
      description: "Conditionally extract detailed information using gpt-5-mini for accuracy"
  estimated_cost_per_call:
    initial_screening: "$0.002 - $0.005"
    detailed_extraction: "$0.01 - $0.03 (varies based on flags)"
    total: "$0.012 - $0.035"
```

Documentation and monitoring metadata:

- `version`, `created` – config versioning information.
- `description` – human‑readable summary of the pipeline.
- `stages` – descriptions of the logical stages, aligned with the step design.
- `estimated_cost_per_call` – rough per‑call LLM cost estimates for budgeting.

Changing these values has **no functional impact**; they are for reference only.

---

## 7. Common modifications and extension patterns

This section shows how to extend or adjust the pipeline using this config pack.

### 7.1 Run the pipeline for a new date

1. Update in `voc_pipeline.yml`:
   - `input.transcript_path` – point to new transcripts.
   - `output.raw_features_path` – new output folder for raw features.
   - `output.combined_table_path` – new path for the combined table (often date‑partitioned).
2. Leave transformation and Databricks configs unchanged unless you need a new target table.

---

### 7.2 Add a new issue type with detailed extraction

1. **Define fields and prompts:**
   - Create a new `fields/<new_issue>_fields.yml` with a `fields:` list describing what to extract.
   - Add a matching `prompts/<new_issue>/system.txt` (and optionally `user.txt`).

2. **Add a new Stage 2 step:**

   ```yaml
   - name: new_issue_detail
     type: voc_structured_extraction   # or voc_list_extraction
     language_model: "gpt-5-mini"
     fields_config: "fields/new_issue_fields.yml"
     templates_path: "prompts/new_issue"
     next_steps: []
   ```

3. **Wire it into Stage 1 routing:**
   - Add an `if_llm_answer` rule under `initial_screening.next_steps` that checks the relevant flag from `initial_screening_fields.yml` and `go_to: [new_issue_detail]` when appropriate.

4. **Expose it in the final table:**
   - If it’s a list step, add an aggregation rule in `aggregation_rules.yml`.
   - Add mappings or merges in `field_mappings.yml` as needed.
   - Append new output columns to `final_columns.yml` and define their `data_types` and optional `column_descriptions`.

---

### 7.3 Add a new field to an existing step

1. Update the relevant `fields/*.yml` file:
   - Add a new `fields:` entry with `name`, `type`, `description`, and `options` (if it’s a literal).
2. If the field should appear in the final table:
   - Add it to `field_mappings.yml`:
     - Either in `source_columns` (if from initial screening).
     - Or via `merge_configs` / aggregation rules (if from a detail table).
   - Add the column to `final_columns.yml` and `data_types`.
3. No change is needed in `voc_pipeline.yml` as long as the step name and `fields_config` path remain the same.

---

### 7.4 Adjust Databricks behavior safely

- For local tests or dry runs:
  - Set `databricks.enable_upsert: false`.
  - You still get the combined Excel at `output.combined_table_path`.
- To move to a long‑term Delta table:
  - Align `databricks.merge_keys` and `partition_by` with `final_columns.yml`.
  - Ensure the target table exists and has compatible schema.

---

By understanding how `voc_pipeline.yml` connects to the supporting configs in `config/voc_config/`, you can confidently modify, extend, and hand over the VOC pipeline without touching application code. Keep paths, field names, and column names consistent across:

- `voc_pipeline.yml` (steps, routing, IO, Databricks),
- `fields/*.yml` and `prompts/*/` (what is extracted and how),
- `field_mappings.yml` and `aggregation_rules.yml` (how raw outputs are combined),
- `final_columns.yml` (final schema and metadata),

to avoid schema drift and ensure smooth runs in both test and production environments.
