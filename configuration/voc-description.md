# Voice of Customer (VOC) Extraction Pipeline

## Overview

The VOC extraction pipeline is a specialized two-stage LLM-based analysis system for insurance renewal calls. It extracts customer insights, renewal intentions, and issue details from call transcripts using a **config-driven, conditional branching architecture**.

### Key Features

- **Two-stage extraction**: Fast screening (gpt-5-mini) → Detailed conditional extraction (gpt-5-mini)
- **Fully config-driven**: All prompts, fields, and logic defined in YAML
- **Dynamic Pydantic models**: Runtime model generation from field definitions
- **Conditional branching**: Extract detailed info only when relevant flags are detected
- **Table aggregation**: Multi-row responses combined into single-row-per-call format
- **Databricks integration**: Automatic upsert to Delta tables with MERGE operations
- **Azure Blob support**: Native storage integration via ArtifactManager

## Architecture

### Pipeline Stages

```
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 1: Initial Screening (gpt-5-mini)                        │
├─────────────────────────────────────────────────────────────────┤
│ Extracts 9 high-level flags:                                    │
│  - renewal_intent (will_renew_but_later, unable_to_renew, etc)  │
│  - competitor_mentioned (yes/no)                                │
│  - pricing_issue (yes/no)                                       │
│  - claims_issue (yes/no)                                        │
│  - payment_issue (yes/no)                                       │
│  - communication_issue (yes/no)                                 │
│  - coverage_gap_mentioned (yes/no)                              │
│  - service_issue (yes/no)                                       │
│  - contact_reason_received (yes/no)                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                            ↓
         ┌──────────────────┴──────────────────┐
         │   Conditional Next Steps            │
         │   (based on flag values)            │
         └──────────────────┬──────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 2: Detailed Extraction (gpt-5-mini)                           │
├─────────────────────────────────────────────────────────────────┤
│ Only runs for calls with specific flags:                        │
│                                                                 │
│ Single-response extractions:                                    │
│  - will_renew_later_detail (if renewal_intent match)            │
│  - unable_to_renew_detail (if renewal_intent match)             │
│  - will_not_renew_detail (if renewal_intent match)              │
│  - undecided_detail (if renewal_intent match)                   │
│  - service_issue_detail (if service_issue=yes)                  │
│                                                                 │
│ Multi-item list extractions:                                    │
│  - competitor_detail (if competitor_mentioned=yes)              │
│  - pricing_issue_detail (if pricing_issue=yes)                  │
│  - claims_issue_detail (if claims_issue=yes)                    │
│  - payment_issue_detail (if payment_issue=yes)                  │
│  - communication_issue_detail (if communication_issue=yes)      │
│  - coverage_gap_detail (if coverage_gap_mentioned=yes)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 3: Table Combination & Transformation                    │
├─────────────────────────────────────────────────────────────────┤
│ Aggregates multi-row responses into single-row-per-call:       │
│  - Custom aggregation functions (find_price, merge_status)     │
│  - Column renames and derivations                              │
│  - Final schema with 44 columns                                │
│  - Partition by call_year, call_month                          │
└─────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│ STAGE 4: Databricks Upsert (optional)                          │
├─────────────────────────────────────────────────────────────────┤
│ MERGE INTO Delta table:                                        │
│  - Target: main.novus_voc.voc_insights                         │
│  - Merge key: filename                                         │
│  - Partitioned by: call_year, call_month                       │
└─────────────────────────────────────────────────────────────────┘
```

### File Structure

```
config/voc_config/
├── voc_pipeline.yml              # Main pipeline configuration
├── field_mappings.yml             # Column mappings and transformations
├── aggregation_rules.yml          # Multi-row aggregation rules
├── final_columns.yml              # Final table schema (44 columns)
├── fields/                        # Field definitions (11 files)
│   ├── initial_screening_fields.yml
│   ├── will_renew_later_fields.yml
│   ├── unable_to_renew_fields.yml
│   ├── will_not_renew_fields.yml
│   ├── undecided_fields.yml
│   ├── competitor_fields.yml
│   ├── pricing_issue_fields.yml
│   ├── claims_issue_fields.yml
│   ├── payment_issue_fields.yml
│   ├── communication_issue_fields.yml
│   ├── coverage_gap_fields.yml
│   └── service_issue_fields.yml
├── prompts/                       # LLM prompt templates (11 dirs)
│   ├── initial_screening/
│   │   ├── system.txt
│   │   └── user.txt
│   ├── competitor/
│   │   ├── system.txt
│   │   └── user.txt
│   └── ... (9 more directories)
└── parameters/                    # Reference data
    ├── insurance_providers.txt    # 32 insurance companies
    └── coverage_gaps.yml          # Coverage types by product
```

## Usage

### Basic Command

```bash
# Run full pipeline with Databricks upsert
novus voc-extract --config config/voc_config/voc_pipeline.yml
```

### Options

```bash
# Skip Databricks upsert (local output only)
novus voc-extract --config voc_pipeline.yml --no-databricks

# Test with limited files
novus voc-extract --config voc_pipeline.yml --max-files 10

# Help
novus voc-extract --help
```

### Configuration

The main configuration file (`voc_pipeline.yml`) controls:

- **Pipeline steps**: All 11 extraction steps with conditional branching
- **Models**: gpt-5-mini for screening, gpt-5-mini for detailed extraction
- **Conditional logic**: `if_llm_answer` rules that trigger downstream steps
- **Output paths**: Raw features, combined table, Databricks target

#### Key Configuration Sections

**Conditional Branching Example:**

```yaml
steps:
  - name: initial_screening
    type: voc_structured_extraction
    language_model: "gpt-5-mini"
    next_steps:
      # Only extract competitor details if flag is yes
      - if_llm_answer:
          field: "competitor_mentioned"
          has_value: "yes"
          go_to: [competitor_detail]

      # Only extract pricing details if flag is yes
      - if_llm_answer:
          field: "pricing_issue"
          has_value: "yes"
          go_to: [pricing_issue_detail]
```

**Databricks Configuration:**

```yaml
databricks:
  catalog: "main"
  schema_name: "novus_voc"
  table_name: "voc_insights"
  merge_keys: ["filename"]
  partition_by: ["call_year", "call_month"]
  enable_upsert: true
```

## Configuration Files

### Field Definitions (`fields/*.yml`)

Each field definition file specifies:

- **Field names and types**: str, int, float, bool, Literal
- **Descriptions**: Help LLM understand what to extract
- **Options**: Literal choices, for example `yes`, `no`, or `not_mentioned`.
- **Required flags**: Enforce non-null responses

**Example (`competitor_fields.yml`):**

```yaml
fields:
  - name: competitor_name
    type: Literal
    description: Name of competitor mentioned
    options_source: ${parameters.insurance_providers}
    required: true

  - name: reason_for_switching
    type: str
    description: Why customer prefers competitor
    required: true

  - name: competitive_advantage
    type: str
    description: What competitor offers that we don't
    required: false
```

### Prompt Templates (`prompts/*/system.txt`, `prompts/*/user.txt`)

Each extraction step has:

- **system.txt**: Expert persona and extraction instructions
- **user.txt**: Transcript template with {transcript} placeholder

Prompts are loaded dynamically at runtime, allowing easy modification without code changes.

### Aggregation Rules (`aggregation_rules.yml`)

Defines how to combine multi-row responses, for example multiple competitors per call:

```yaml
aggregations:
  - table_name: competitor
    groupby: [filename]
    aggregations:
      competitor_name:
        function: custom
        custom_function: list_unique
      price_quoted:
        function: custom
        custom_function: find_price
```

**Available Custom Functions:**

- `list_unique`: Comma-separated unique values
- `find_price`: Extract first numeric price
- `merge_claim_reimbursement_status`: Combine claim status info
- `extract_mobile_number`: Find valid mobile numbers
- `extract_renewal_date`: Parse date mentions
- `merge_reason_for_switching`: Deduplicate switching reasons
- `merge_competitive_advantage`: Combine advantages
- `merge_recommended_actions`: Consolidate action items

### Final Schema (`final_columns.yml`)

Specifies the 44-column output table:

```yaml
columns:
  - filename
  - call_year
  - call_month
  - renewal_intent
  - competitor_name
  - pricing_issue_type
  - claim_details
  # ... 37 more columns

primary_key: filename
partition_by: [call_year, call_month]

data_types:
  call_year: int
  call_month: int
  # ... type specifications
```

## Extensions

### VocStructuredExtractionExtension

Handles **single-response extractions** (one result per call):

- Dynamically builds Pydantic models from field definitions
- Loads dynamic options from external files (insurance providers, etc.)
- Uses LiteLLM with `response_format` for structured JSON output
- Prefixes field names with step name to avoid collisions

**Config:**

```yaml
- name: will_renew_later_detail
  type: voc_structured_extraction
  language_model: "gpt-5-mini"
  fields_config: "fields/will_renew_later_fields.yml"
  templates_path: "prompts/will_renew_later"
```

### VocListExtractionExtension

Handles **multi-item extractions** (multiple results per call):

- Creates list wrapper models, for example `CompetitorList`
- Each item is a full Pydantic model with validation
- Supports `max_items` limit to prevent runaway extraction

**Config:**

```yaml
- name: competitor_detail
  type: voc_list_extraction
  language_model: "gpt-5-mini"
  fields_config: "fields/competitor_fields.yml"
  templates_path: "prompts/competitor"
  max_items: 10
```

### VocTableCombiner

Transforms raw extraction outputs into final schema:

1. **Load aggregation rules** from YAML
2. **Group by filename** and apply custom functions
3. **Apply field mappings** (renames, derivations)
4. **Reorder columns** per final schema
5. **Output** to Excel, CSV, or Parquet

## Cost Estimation

**Per Call Cost:**

- Initial screening: $0.002 - $0.005 (gpt-5-mini)
- Detailed extraction: $0.01 - $0.03 (gpt-5-mini, varies by flags)
- **Total: $0.012 - $0.035 per call**

**1000 Call Batch:**

- Estimated cost: $12 - $35
- Processing time: ~5-10 minutes (with concurrency)

**Cost Optimization:**

- Use gpt-5-mini for screening (80% cheaper)
- Conditional extraction reduces unnecessary API calls
- Only process calls with relevant issues

## Output

### Raw Features

`<output-dir>/voc_features/raw/`

- One JSON file per transcript per step
- Contains raw LLM responses with all extracted fields

### Combined Table

`<output-dir>/voc_features/combined/voc_combined.xlsx`

- Single-row-per-call format
- 44 columns with aggregated multi-row data
- Partitioned by call_year, call_month

### Databricks Table

`main.novus_voc.voc_insights`

- Delta table with MERGE INTO upserts
- Partitioned for query performance
- Primary key: filename

## Development

### Adding New Fields

1. **Update field definition YAML** (`fields/<step>_fields.yml`)
2. **Update aggregation rules** if multi-row field (`aggregation_rules.yml`)
3. **Update final columns** if output field (`final_columns.yml`)
4. **Test with sample transcript**

### Adding New Extraction Step

1. **Create field definition**: `fields/new_step_fields.yml`
2. **Create prompts**: `prompts/new_step/system.txt`, `prompts/new_step/user.txt`
3. **Add step to pipeline**: Update `voc_pipeline.yml` with step config
4. **Add conditional trigger**: Add `if_llm_answer` in initial_screening next_steps
5. **Update aggregation**: Add rules if multi-row extraction
6. **Test end-to-end**

### Modifying Prompts

Simply edit text files in `prompts/` directories - no code changes required.

```bash
# Edit system prompt for competitor extraction
vim config/voc_config/prompts/competitor/system.txt

# Re-run pipeline (prompts loaded dynamically)
novus voc-extract --config config/voc_config/voc_pipeline.yml
```

## Testing

### Test with Sample Transcripts

```bash
# Test with 10 files
novus voc-extract --config config/voc_config/voc_pipeline.yml --max-files 10 --no-databricks

# Check outputs in your configured output directory
```

### Validate Extraction Quality

```python
import pandas as pd

# Load combined table (use your configured output path)
df = pd.read_excel("<output-dir>/voc_features/combined/voc_combined.xlsx")

# Check coverage
print(f"Total calls: {len(df)}")
print(f"Competitor mentions: {df['competitor_mentioned'].value_counts()}")
print(f"Renewal intents: {df['renewal_intent'].value_counts()}")

# Spot check specific calls
sample = df[df['competitor_mentioned'] == 'yes'].head(5)
print(sample[['filename', 'competitor_name', 'reason_for_switching']])
```

## Troubleshooting

### Common Issues

**1. Missing prompt templates**

```
Error: Template file not found: prompts/competitor/system.txt
```

**Solution**: Ensure all prompt files exist in `prompts/` subdirectories

**2. Dynamic options not loading**

```
Error: Cannot load options from ${parameters.insurance_providers}
```

**Solution**: Check `parameters/insurance_providers.txt` exists and is readable

**3. Databricks connection failed**

```
Error: Databricks operation failed
```

**Solution**: Verify environment variables:

- `DATABRICKS_SERVER_HOSTNAME`
- `DATABRICKS_ACCESS_TOKEN`
- `DATABRICKS_WAREHOUSE_ID`

**4. Aggregation function error**

```
Error: Custom function 'find_price' not found
```

**Solution**: Check `aggregation_rules.yml` custom_functions section

### Debug Mode

Set logging level in `.env`:

```bash
LOG_LEVEL=DEBUG
```

Re-run with verbose output:

```bash
novus voc-extract --config voc_pipeline.yml
```

## Best Practices

1. **Test incrementally**: Start with 10 files, validate outputs, scale up
2. **Monitor costs**: Track LLM API usage per batch
3. **Version prompts**: Keep prompt templates in version control
4. **Validate field changes**: Run regression tests after config updates
5. **Partition data**: Use call_year/call_month for efficient queries
6. **Review edge cases**: Manually check calls with unusual extraction patterns

## Integration with Existing Novus Pipelines

The VOC pipeline is a **standalone extraction pipeline** that can run independently or alongside existing Novus pipelines:

```bash
# Run transcription first
novus transcribe --config transcribe_config.yml

# Then run VOC extraction on transcripts
novus voc-extract --config voc_pipeline.yml
```

Uses same infrastructure:

- **ArtifactManager** for storage (S3, Azure Blob, local)
- **LiteLLM** for model abstraction
- **DatabricksTableManager** for Delta table operations
- **ExtractionPipeline** for orchestration

## Future Enhancements

- [ ] Implement confidence scores for extracted fields
- [ ] Add data quality metrics (extraction coverage, field completeness)
- [ ] Create dashboard for VOC insights visualization
- [ ] Add support for custom post-processing hooks
- [ ] Implement retry logic for failed LLM calls
- [ ] Add export to PowerBI/Tableau formats

## References

- **Main codebase**: `src/novus_calls/`
- **Extensions**: `src/novus_calls/core/extraction/extensions/voc_*/`
- **Combiners**: `src/novus_calls/core/combiners/voc_table_combiner.py`
- **CLI**: `src/novus_calls/cli/commands/voc_extract.py`
- **Models**: `src/novus_calls/core/extraction/voc_models.py`
