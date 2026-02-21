---
layout: default
title: Analysis Patterns
nav_order: 4
---

# Common Analysis Patterns
{: .no_toc }

Workflows and code patterns for ICU research with CLIF.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Creating Wide Datasets

CLIF stores data in long format (one row per observation). For many analyses, you'll want wide format (one row per patient-hour):

```python
from clifpy import ClifOrchestrator

clif = ClifOrchestrator(data_dir="path/to/clif/data")
clif.load_tables(["vitals", "labs", "respiratory_support"])

# Create hourly wide dataset
wide_df = clif.create_wide_dataset(
    tables=["vitals", "labs"],
    time_resolution="1h",
    aggregation="mean"  # or "last", "first", "max", "min"
)

# Result: one row per hospitalization_id + hour
# Columns: heart_rate_mean, sbp_mean, creatinine_mean, etc.
```

### Custom Aggregations

```python
import polars as pl

wide_df = clif.create_wide_dataset(
    tables=["vitals"],
    time_resolution="1h",
    aggregation={
        "heart_rate": ["mean", "max"],
        "sbp": ["mean", "min"],
        "spo2": ["mean", "min"]
    }
)
# Result: heart_rate_mean, heart_rate_max, sbp_mean, sbp_min, etc.
```

---

## Clinical Scores

### SOFA Score

The Sequential Organ Failure Assessment score quantifies organ dysfunction:

```python
# Calculate SOFA scores
sofa_df = clif.calculate_sofa_scores()

# Daily worst SOFA
daily_sofa = sofa_df.group_by([
    "hospitalization_id",
    pl.col("score_dttm").dt.date().alias("date")
]).agg(
    pl.col("sofa_total").max().alias("sofa_max"),
    pl.col("sofa_respiratory").max(),
    pl.col("sofa_coagulation").max(),
    pl.col("sofa_liver").max(),
    pl.col("sofa_cardiovascular").max(),
    pl.col("sofa_cns").max(),
    pl.col("sofa_renal").max()
)
```

{: .note }
clifpy's SOFA calculation includes SpO2→PaO2 imputation when PaO2 is unavailable, using the validated method from the CLIF concept paper.

### Comorbidity Indices

```python
# Charlson Comorbidity Index
charlson_df = clif.calculate_charlson_index()

# Elixhauser Comorbidities
elix_df = clif.calculate_elixhauser()
```

---

## Cohort Identification

### ICU Encounters

```python
# Identify ICU stays using ADT location categories
adt = clif.adt

icu_stays = adt.filter(
    pl.col("location_category").is_in([
        "icu", "cardiac_icu", "neuro_icu", "burn_icu"
    ])
)

# Get unique ICU encounters
icu_encounters = icu_stays.select("hospitalization_id").unique()
```

### Ventilated Patients

```python
resp = clif.respiratory_support

# Patients on invasive mechanical ventilation
imv_patients = resp.filter(
    pl.col("device_category") == "IMV"
).select("hospitalization_id").unique()

# First intubation time per encounter
first_intubation = (
    resp
    .filter(pl.col("device_category") == "IMV")
    .group_by("hospitalization_id")
    .agg(pl.col("recorded_dttm").min().alias("intubation_time"))
)
```

### Vasopressor Use

```python
meds = clif.medication_admin_continuous

# Patients on vasopressors
vasopressor_categories = [
    "norepinephrine", "epinephrine", "vasopressin", 
    "phenylephrine", "dopamine"
]

vasopressor_patients = (
    meds
    .filter(pl.col("med_category").is_in(vasopressor_categories))
    .select("hospitalization_id")
    .unique()
)
```

---

## Encounter Stitching

Sometimes a single ICU stay spans multiple `hospitalization_id` values (e.g., patient transferred between units). clifpy can stitch these together:

```python
# Stitch encounters within 24-hour windows
clif.stitch_encounters(
    time_window_hours=24,
    same_patient_only=True
)

# Access stitched encounter IDs
stitched_df = clif.stitched_encounters
# Contains: original_hospitalization_id, stitched_encounter_id
```

---

## Time-Relative Analysis

Many ICU analyses are relative to a key event (intubation, ICU admission, etc.):

```python
# Get first ICU admission time per encounter
icu_admit = (
    adt
    .filter(pl.col("location_category").str.contains("icu"))
    .group_by("hospitalization_id")
    .agg(pl.col("in_dttm").min().alias("icu_admit_time"))
)

# Join with vitals and calculate hours since ICU admission
vitals_relative = (
    clif.vitals
    .join(icu_admit, on="hospitalization_id")
    .with_columns(
        ((pl.col("recorded_dttm") - pl.col("icu_admit_time"))
         .dt.total_hours()
         .alias("hours_since_icu_admit"))
    )
)

# Filter to first 48 hours
first_48h = vitals_relative.filter(
    pl.col("hours_since_icu_admit").is_between(0, 48)
)
```

---

## Federated Analysis Pattern

For multi-site CLIF projects, structure your code to run identically at each site:

```python
# analysis.py - runs at each site
from clifpy import ClifOrchestrator
import polars as pl

def run_analysis(data_dir: str, output_dir: str):
    """
    Site-level analysis that produces aggregate results only.
    No patient-level data leaves the site.
    """
    clif = ClifOrchestrator(data_dir=data_dir)
    clif.load_tables(["patient", "hospitalization", "vitals"])
    
    # Aggregate statistics (safe to share)
    results = {
        "n_patients": clif.patient.n_unique("patient_id"),
        "n_encounters": clif.hospitalization.height,
        "mean_age": clif.patient["age"].mean(),
        "vitals_summary": (
            clif.vitals
            .group_by("vital_category")
            .agg([
                pl.col("vital_value").mean().alias("mean"),
                pl.col("vital_value").std().alias("std"),
                pl.col("vital_value").count().alias("n")
            ])
        )
    }
    
    # Save aggregates
    pl.DataFrame(results["vitals_summary"]).write_csv(
        f"{output_dir}/vitals_summary.csv"
    )
    
    return results

if __name__ == "__main__":
    import sys
    run_analysis(sys.argv[1], sys.argv[2])
```

---

## Table 1 Generation

Create descriptive statistics tables using [CLIF-TableOne](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-TableOne):

```python
from clif_tableone import TableOne

# Define variables
categorical = ["sex", "race", "ethnicity"]
continuous = ["age", "sofa_admission"]

# Create Table 1
table1 = TableOne(
    data=cohort_df,
    categorical=categorical,
    continuous=continuous,
    groupby="mortality_90d",  # optional stratification
    pval=True
)

# Export
table1.to_csv("table1.csv")
table1.to_latex("table1.tex")
```

---

## Performance Tips

### Use Lazy Evaluation

```python
# Instead of loading everything
df = pl.read_parquet("huge_file.parquet")  # Loads into RAM
filtered = df.filter(...)

# Use lazy evaluation
df = pl.scan_parquet("huge_file.parquet")  # Just a plan
filtered = df.filter(...).collect()  # Executes optimized query
```

### Partition by Encounter

For very large datasets, consider partitioning:

```python
# Process in chunks by encounter
encounter_ids = clif.hospitalization["hospitalization_id"].unique().to_list()

for chunk in chunked(encounter_ids, 1000):
    subset = clif.vitals.filter(
        pl.col("hospitalization_id").is_in(chunk)
    )
    # Process subset...
```

---

## Next Steps

- [Use the project template]({{ site.baseurl }}/project-template/) for reproducible, shareable research
- [Review best practices]({{ site.baseurl }}/best-practices/) to avoid common pitfalls
