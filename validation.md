---
layout: default
title: Validation
nav_order: 3
---

# Data Validation
{: .no_toc }

Ensure your CLIF data meets quality standards before analysis.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Validation Matters

In federated research, data quality at each site directly impacts the reliability of multi-site findings. CLIF provides standardized validation to ensure:

- **Schema compliance** - Required columns exist with correct types
- **Vocabulary adherence** - Categories match mCIDE controlled vocabularies  
- **Clinical plausibility** - Values fall within physiologically reasonable ranges
- **Temporal consistency** - Timestamps make sense (no future dates, proper ordering)

---

## Quick Validation

```python
from clifpy import ClifOrchestrator

clif = ClifOrchestrator(data_dir="path/to/clif/data")

# Load with validation enabled
clif.load_tables(
    tables=["vitals", "labs"],
    validate=True
)

# Check validation results
print(clif.validation_report)
```

---

## Schema Validation

CLIF tables must conform to YAML schema definitions. The validator checks:

### Required Columns

```python
from clifpy.validator import SchemaValidator

validator = SchemaValidator()

# Validate a single table
results = validator.validate_table(
    df=vitals_df,
    table_name="vitals"
)

if not results.is_valid:
    print("Missing required columns:")
    for col in results.missing_columns:
        print(f"  - {col}")
```

### Data Types

```python
# Check data type mismatches
for col, expected, actual in results.type_mismatches:
    print(f"{col}: expected {expected}, got {actual}")
```

{: .warning }
**Common pitfall:** DateTime columns must be actual datetime types, not strings. Use `pl.col("recorded_dttm").str.to_datetime()` if needed.

---

## Vocabulary Validation

mCIDE (minimal Common ICU Data Elements) defines controlled vocabularies for categorical fields.

### Checking Categories

```python
from clifpy.validator import VocabularyValidator

vocab_validator = VocabularyValidator()

# Validate vital categories against mCIDE
results = vocab_validator.validate_categories(
    df=vitals_df,
    column="vital_category",
    vocabulary="vitals"
)

# See invalid values
if results.invalid_values:
    print("Invalid vital categories found:")
    for val, count in results.invalid_values.items():
        print(f"  '{val}': {count} records")
        
    # Fuzzy match suggestions
    for val in results.invalid_values:
        suggestion = vocab_validator.suggest_match(val, "vitals")
        print(f"  '{val}' -> did you mean '{suggestion}'?")
```

### Valid Categories Reference

| Table | Category Column | # Valid Values |
|:------|:----------------|:---------------|
| vitals | vital_category | 9 |
| labs | lab_category | 52 |
| medication_admin_continuous | med_category | 91 |
| respiratory_support | device_category | 9 |
| position | position_category | 6 |

{: .note }
Find the full mCIDE vocabulary files in the [CLIF skills repo](https://github.com/Common-Longitudinal-ICU-data-Format/skills/tree/main/skills/clif-icu/mCIDE).

---

## Clinical Plausibility

CLIF Lighthouse defines clinically reasonable thresholds:

### Vital Sign Ranges

| Vital | Lower Bound | Upper Bound |
|:------|:------------|:------------|
| temp_c | 32 | 44 |
| heart_rate | 0 | 300 |
| sbp | 0 | 300 |
| dbp | 0 | 200 |
| spo2 | 50 | 100 |
| respiratory_rate | 0 | 60 |

### Lab Value Ranges

| Lab | Lower Bound | Upper Bound |
|:----|:------------|:------------|
| creatinine | 0 | 20 |
| hemoglobin | 2 | 25 |
| sodium | 90 | 210 |
| potassium | 0 | 15 |
| lactate | 0 | 30 |
| platelets | 0 | 2000 |

### Detecting Outliers

```python
from clifpy.validator import OutlierDetector

detector = OutlierDetector()

# Find outliers in vitals
outliers = detector.detect_outliers(
    df=vitals_df,
    table_name="vitals"
)

print(f"Found {len(outliers)} outlier records")
print(f"By category:\n{outliers.group_by('vital_category').len()}")
```

### Handling Outliers

```python
# Option 1: Flag outliers (recommended for transparency)
vitals_df = vitals_df.with_columns(
    detector.flag_outliers(vitals_df, "vitals").alias("is_outlier")
)

# Option 2: Cap at thresholds
vitals_df = detector.cap_outliers(vitals_df, "vitals")

# Option 3: Remove outliers (use cautiously)
vitals_clean = vitals_df.filter(~pl.col("is_outlier"))
```

---

## Temporal Validation

### No Future Timestamps

```python
from datetime import datetime

now = datetime.now()

future_records = vitals_df.filter(
    pl.col("recorded_dttm") > now
)

if len(future_records) > 0:
    print(f"⚠️ Found {len(future_records)} records with future timestamps")
```

### ADT Time Overlaps

For ADT (Admission-Discharge-Transfer) records, times shouldn't overlap:

```python
from clifpy.validator import TemporalValidator

temporal = TemporalValidator()

# Check for overlapping ADT records
overlaps = temporal.find_adt_overlaps(adt_df)

if len(overlaps) > 0:
    print(f"⚠️ Found {len(overlaps)} overlapping ADT records")
```

---

## CLIF Lighthouse

For interactive validation, use [CLIF Lighthouse](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Lighthouse) - a Streamlit-based web app:

```bash
# Install
pip install clif-lighthouse

# Run
clif-lighthouse --data-dir path/to/clif/data
```

Lighthouse provides:
- Visual distribution plots
- Interactive outlier exploration
- Downloadable QC reports
- Category validation with fuzzy matching

---

## Validation Checklist

Before running analyses or sharing results, confirm:

- [ ] All required columns present
- [ ] Data types correct (especially datetimes)
- [ ] Categories match mCIDE vocabularies
- [ ] No impossible values (negative ages, SpO2 > 100)
- [ ] Outliers flagged or handled consistently
- [ ] No future timestamps
- [ ] ADT records don't overlap

---

## Next Steps

With validated data, you're ready to:
- [Perform common analyses]({{ site.baseurl }}/analysis/) 
- [Calculate clinical scores]({{ site.baseurl }}/analysis/#clinical-scores)
