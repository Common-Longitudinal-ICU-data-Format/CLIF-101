---
layout: default
title: Loading Data
nav_order: 2
---

# Loading CLIF Data
{: .no_toc }

Learn how to efficiently load and explore CLIF tables using clifpy.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The ClifOrchestrator

The `ClifOrchestrator` is your central hub for working with CLIF data. It handles:
- Loading all 18 CLIF tables with consistent configuration
- Schema validation against mCIDE vocabularies
- Encounter stitching (linking related ICU stays)
- Wide dataset creation
- Clinical score computation (SOFA, comorbidities)

### Basic Setup

```python
from clifpy import ClifOrchestrator

# Point to your CLIF data directory
clif = ClifOrchestrator(
    data_dir="path/to/clif/data",
    file_format="parquet"  # or "csv"
)

# Load specific tables
clif.load_tables(["patient", "hospitalization", "vitals", "labs", "adt"])

# Access loaded data
patients = clif.patient
vitals = clif.vitals
```

{: .note }
clifpy uses DuckDB and Polars under the hood, so it can handle datasets much larger than your RAM.

---

## CLIF Table Overview

CLIF organizes ICU data into logical categories:

### Demographics
| Table | Description |
|:------|:------------|
| `patient` | Patient demographics (age, sex, race, ethnicity) |
| `hospitalization` | Hospital encounters with admission/discharge times |

### Location & Movement
| Table | Description |
|:------|:------------|
| `adt` | Admission-Discharge-Transfer events with location details |

### Clinical Data
| Table | Description |
|:------|:------------|
| `vitals` | Vital signs (HR, BP, SpO2, temp, RR) |
| `labs` | Laboratory results (52 categories) |
| `respiratory_support` | Ventilator settings and oxygen delivery |
| `position` | Patient positioning (prone, supine, etc.) |

### Medications
| Table | Description |
|:------|:------------|
| `medication_admin_continuous` | Continuous infusions (vasopressors, sedatives) |
| `medication_admin_intermittent` | Intermittent/bolus medications |

### Specialized
| Table | Description |
|:------|:------------|
| `crrt_therapy` | Continuous renal replacement therapy |
| `microbiology_culture` | Culture results and sensitivities |
| `code_status` | Goals of care documentation |

---

## Loading Individual Tables

For more control, load tables individually:

```python
from clifpy import VitalsTable, LabsTable

# Load vitals with validation
vitals = VitalsTable(
    data_dir="path/to/clif/data",
    validate=True  # Check against mCIDE vocabularies
)
vitals.load()

# Explore the data
print(f"Records: {len(vitals.df):,}")
print(f"Unique encounters: {vitals.df['hospitalization_id'].n_unique():,}")
print(f"Date range: {vitals.df['recorded_dttm'].min()} to {vitals.df['recorded_dttm'].max()}")
```

---

## Filtering Data

### By Time Window

```python
import polars as pl
from datetime import datetime

# Filter to 2024 data
vitals_2024 = vitals.df.filter(
    pl.col("recorded_dttm").dt.year() == 2024
)
```

### By Encounter

```python
# Get vitals for specific encounters
encounter_ids = ["ENC001", "ENC002", "ENC003"]
subset = vitals.df.filter(
    pl.col("hospitalization_id").is_in(encounter_ids)
)
```

### By Vital Category

```python
# Get only heart rate and blood pressure
hr_bp = vitals.df.filter(
    pl.col("vital_category").is_in(["heart_rate", "sbp", "dbp"])
)
```

---

## Handling Large Datasets

clifpy is designed for large datasets. Here are some tips:

### Lazy Loading

```python
# Use lazy evaluation for large files
vitals_lazy = pl.scan_parquet("path/to/vitals.parquet")

# Apply filters before collecting
filtered = (
    vitals_lazy
    .filter(pl.col("vital_category") == "heart_rate")
    .filter(pl.col("vital_value").is_between(40, 200))
    .collect()  # Only now does it execute
)
```

### Memory-Efficient Aggregation

```python
# Aggregate without loading full dataset
hourly_means = (
    vitals_lazy
    .group_by([
        "hospitalization_id",
        pl.col("recorded_dttm").dt.truncate("1h").alias("hour"),
        "vital_category"
    ])
    .agg(pl.col("vital_value").mean().alias("mean_value"))
    .collect()
)
```

---

## Next Steps

Now that you can load data, learn how to:
- [Validate your data]({{ site.baseurl }}/validation/) against CLIF schemas
- [Perform common analyses]({{ site.baseurl }}/analysis/) with CLIF
- [Use the project template]({{ site.baseurl }}/project-template/) for reproducible research
