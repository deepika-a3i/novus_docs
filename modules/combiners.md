# Combiners

Combiners fold step-level outputs into a single, structured table. The primary combiner in this repo is the VOC table combiner.

## VOC Table Combiner

- File: `src/novus_calls/core/combiners/voc_table_combiner.py`
- Inputs: raw step outputs, aggregation rules, and field mappings.
- Outputs: a single row per call with normalized columns.

### Responsibilities

- Load raw extraction outputs per step.
- Apply `aggregation_rules.yml` to list outputs.
- Merge structured outputs into the main table.
- Rename, derive, and reorder columns based on `field_mappings.yml` and `final_columns.yml`.
- Write the combined table to Excel, CSV, or Parquet.

### Custom aggregation

Custom functions such as `find_price`, `list_unique`, and `merge_reason_for_switching` are defined in the aggregation rules file and executed by the combiner.

If you add a new list-type VOC step, you will almost always update the combiner configuration to fold it into the final table.
