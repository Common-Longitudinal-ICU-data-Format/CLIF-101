---
layout: default
title: Home
nav_order: 1
description: "CLIF 101 - A gentle introduction to coding with CLIF datasets"
permalink: /
---

# Welcome to CLIF 101 🏥
{: .fs-9 }

A gentle introduction to coding with established CLIF datasets. Learn best practices for analysis, validation, and collaboration.
{: .fs-6 .fw-300 }

[Get Started](#getting-started){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[View clifpy Docs](https://github.com/Common-Longitudinal-ICU-data-Format/clifpy){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What is CLIF?

The **Common Longitudinal ICU data Format (CLIF)** is a federated data standard designed specifically for critical care research. Unlike generic common data models, CLIF preserves the temporal granularity essential for ICU research while enabling privacy-preserving multi-site collaboration.

### Why CLIF matters

| Challenge | CLIF Solution |
|:----------|:--------------|
| Data silos across hospitals | Federated model - data stays local |
| Lost temporal granularity | Hourly resolution preserved |
| Inconsistent vocabularies | mCIDE controlled vocabularies |
| Slow multi-site studies | Days instead of years |

---

## Getting Started

This tutorial assumes you have access to a CLIF dataset at your institution. If you're new to CLIF and need to build your dataset first, check out:
- [CLIF Data Dictionary](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF) - Schema definitions
- [EHR-TO-CLIF](https://github.com/Common-Longitudinal-ICU-data-Format/EHR-TO-CLIF) - ETL examples from various sites

### Prerequisites

- Python 3.9+
- Access to CLIF-formatted data (Parquet or CSV files)
- Basic familiarity with pandas/polars

### Install clifpy

```bash
pip install clifpy
```

{: .note }
clifpy uses DuckDB and Polars under the hood for memory-efficient processing of large datasets.

---

## Quick Example

Here's a taste of what working with CLIF looks like:

```python
from clifpy import ClifOrchestrator

# Initialize with your CLIF data directory
clif = ClifOrchestrator(data_dir="path/to/clif/data")

# Load core tables
clif.load_tables(["patient", "hospitalization", "vitals", "labs"])

# Create an hourly wide dataset for analysis
wide_df = clif.create_wide_dataset(
    tables=["vitals", "labs"],
    time_resolution="1h"
)

# Calculate SOFA scores
sofa_df = clif.calculate_sofa_scores()
```

---

## What You'll Learn

<div class="grid-container">
  <div class="grid-item">
    <h3>📂 Loading Data</h3>
    <p>Use clifpy to efficiently load and validate CLIF tables</p>
  </div>
  <div class="grid-item">
    <h3>✅ Validation</h3>
    <p>Quality control your data against the CLIF schema</p>
  </div>
  <div class="grid-item">
    <h3>📊 Analysis Patterns</h3>
    <p>Common workflows for ICU research</p>
  </div>
  <div class="grid-item">
    <h3>🔧 Project Template</h3>
    <p>Structure your work for federated collaboration</p>
  </div>
</div>

---

## Ready to dive in?

Start with [Loading CLIF Data]({{ site.baseurl }}/loading-data/) to learn the fundamentals.

{: .highlight }
**Pro tip:** Always read the schema files before writing code! Add this to your workflow: check `clifpy`'s schema definitions to get exact column names.
