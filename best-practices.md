---
layout: default
title: Best Practices
nav_order: 6
---

# Best Practices
{: .no_toc }

Tips, gotchas, and lessons learned from the CLIF community.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 🎯 Always Read the Schema First

The #1 source of bugs in CLIF projects: **guessing column names instead of checking the schema**.

{: .warning }
Don't assume column names! Check the [CLIF data dictionary](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF) or clifpy schemas before writing code.

```python
# ❌ Bad - guessing the column name
df.select("procedure_dttm")  # Wrong!

# ✅ Good - check the schema first
# The actual column is procedure_billed_dttm
df.select("procedure_billed_dttm")
```

**Pro tip:** Add to your AGENTS.md or Claude skill:
> "Always read schema files before writing code that uses a CLIF table"

---

## 🔒 Data Security

### Never Commit Patient Data

```bash
# Your .gitignore should ALWAYS include:
data/
*.csv
*.parquet
```

### Only Share Aggregates

```python
# ❌ Bad - sharing patient-level data
results = df.select("patient_id", "outcome")

# ✅ Good - sharing only aggregates
results = df.group_by("site").agg([
    pl.count().alias("n"),
    pl.col("outcome").mean().alias("mortality_rate")
])
```

### Minimum Cell Sizes

For any summary statistics, ensure minimum cell sizes (typically n ≥ 10) to prevent re-identification:

```python
def safe_aggregate(df, group_cols, agg_cols, min_n=10):
    """Only return aggregates with sufficient sample size."""
    result = df.group_by(group_cols).agg(agg_cols)
    return result.filter(pl.col("n") >= min_n)
```

---

## 📊 Working with Vitals

### Use the Right Categories

Check mCIDE for valid `vital_category` values:

| Category | Description |
|:---------|:------------|
| `heart_rate` | Heart rate (bpm) |
| `sbp` | Systolic blood pressure |
| `dbp` | Diastolic blood pressure |
| `map` | Mean arterial pressure |
| `respiratory_rate` | Respiratory rate |
| `spo2` | Oxygen saturation |
| `temp_c` | Temperature (Celsius) |
| `height_cm` | Height |
| `weight_kg` | Weight |

### Handle Missing Data Explicitly

```python
# ❌ Bad - silently dropping missing values
mean_hr = df.filter(pl.col("vital_category") == "heart_rate")["vital_value"].mean()

# ✅ Good - explicit about missingness
hr_data = df.filter(pl.col("vital_category") == "heart_rate")
n_missing = hr_data.filter(pl.col("vital_value").is_null()).height
n_total = hr_data.height
print(f"Heart rate missing: {n_missing}/{n_total} ({100*n_missing/n_total:.1f}%)")
mean_hr = hr_data["vital_value"].drop_nulls().mean()
```

---

## 💊 Working with Medications

### Understand Continuous vs. Intermittent

| Table | Use Case | Key Fields |
|:------|:---------|:-----------|
| `medication_admin_continuous` | Drips, infusions | `admin_dose`, `dose_unit`, `start_dttm`, `end_dttm` |
| `medication_admin_intermittent` | Boluses, pills | `admin_dose`, `dose_unit`, `admin_dttm` |

### Weight-Based Dosing

Many ICU medications are dosed per kg. Always clarify:

```python
# Check if dose is weight-based
if "mcg/kg" in row["dose_unit"] or "mg/kg" in row["dose_unit"]:
    # Dose is already weight-normalized
    dose_per_kg = row["admin_dose"]
else:
    # Need to normalize by weight
    dose_per_kg = row["admin_dose"] / patient_weight_kg
```

### Vasopressor Equivalents

When comparing vasopressor intensity across patients:

```python
# Norepinephrine equivalent calculations
NEE_FACTORS = {
    "norepinephrine": 1.0,
    "epinephrine": 1.0,
    "dopamine": 0.01,  # mcg/kg/min to mcg/kg/min NEE
    "phenylephrine": 0.1,
    "vasopressin": 2.5  # units/min to mcg/kg/min NEE (approximate)
}
```

---

## 🫁 Working with Respiratory Support

### Device Category Hierarchy

CLIF defines a severity hierarchy for respiratory support:

```
IMV > NIPPV > CPAP > High Flow NC > Face Mask > Trach Collar > Nasal Cannula > Room Air
```

### Identifying Ventilated Patients

```python
# ❌ Incomplete - misses NIV
imv_only = df.filter(pl.col("device_category") == "IMV")

# ✅ Better - includes all mechanical ventilation
mechanical_vent = df.filter(
    pl.col("device_category").is_in(["IMV", "NIPPV", "CPAP"])
)

# ✅ Best - document your definition
INVASIVE_VENT = ["IMV"]
NONINVASIVE_VENT = ["NIPPV", "CPAP"]
ALL_MECHANICAL_VENT = INVASIVE_VENT + NONINVASIVE_VENT
```

---

## ⏰ Time Zone Handling

### Be Explicit About Time Zones

```python
# ❌ Bad - ambiguous timezone
admit_time = df["admit_dttm"][0]

# ✅ Good - explicit timezone handling
import polars as pl
from zoneinfo import ZoneInfo

# Convert to site's local timezone for display
local_tz = ZoneInfo("America/Chicago")
admit_time = df["admit_dttm"][0].replace(tzinfo=local_tz)
```

### Consistent UTC Storage

CLIF stores timestamps in the site's local time. When combining data across sites, convert to UTC:

```python
# Standardize to UTC for cross-site analysis
df = df.with_columns(
    pl.col("recorded_dttm")
    .dt.replace_time_zone("America/Chicago")
    .dt.convert_time_zone("UTC")
    .alias("recorded_dttm_utc")
)
```

---

## 🧪 Testing with Synthetic Data

Before running on real data, test with [synthetic CLIF](https://github.com/AartikSarma/synthetic_clif):

```python
# In tests/conftest.py
import pytest
from synthetic_clif import generate_clif_dataset

@pytest.fixture
def synthetic_clif():
    """Generate synthetic CLIF data for testing."""
    return generate_clif_dataset(
        n_patients=100,
        n_encounters=150,
        seed=42
    )
```

{: .note }
When you find bugs in synthetic_clif, contribute back! Open an issue or PR at the repo.

---

## 📝 Documentation Checklist

Before sharing your project:

- [ ] README with clear setup instructions
- [ ] Analysis plan (preferably pre-registered)
- [ ] Data dictionary for derived variables
- [ ] Requirements.txt with pinned versions
- [ ] Example config file (without sensitive paths)
- [ ] Comments explaining non-obvious code

---

## 🐛 Common Errors

### "Column not found"

```python
# Usually means you're using wrong column name
# Check the schema!
```

### DateTime parsing errors

```python
# Ensure datetime columns are actually datetime type
df = df.with_columns(
    pl.col("recorded_dttm").str.to_datetime("%Y-%m-%d %H:%M:%S")
)
```

### Memory errors with large files

```python
# Use lazy evaluation
df = pl.scan_parquet("large_file.parquet")
result = df.filter(...).select(...).collect()
```

### Validation failures

```python
# Check which categories don't match mCIDE
invalid = df.filter(
    ~pl.col("vital_category").is_in(VALID_VITAL_CATEGORIES)
)
print(invalid["vital_category"].unique())
```

---

## 🆘 Getting Help

- **#clif-code-ecosystem** - Slack channel for coding questions
- **#clifpy** - Slack channel for clifpy-specific issues
- **GitHub Issues** - For bugs in clifpy, CLIF-MIMIC, or other repos
- **Weekly CLIF Calls** - Thursdays 2-3 PM CT

---

## Contributing Back

Found a bug? Improved a workflow? Please contribute!

1. **Documentation improvements** - PR to this repo or the main CLIF repo
2. **clifpy enhancements** - PR to [clifpy](https://github.com/Common-Longitudinal-ICU-data-Format/clifpy)
3. **ETL patterns** - Add your site's approach to [EHR-TO-CLIF](https://github.com/Common-Longitudinal-ICU-data-Format/EHR-TO-CLIF)
4. **Validation rules** - Improve CLIF Lighthouse detection

---

*Have a tip that should be here? Let us know in #clif-code-ecosystem!*
