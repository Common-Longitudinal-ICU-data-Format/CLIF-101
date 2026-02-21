---
layout: default
title: Project Template
nav_order: 5
---

# Using the CLIF Project Template
{: .no_toc }

Structure your CLIF research for reproducibility and federated collaboration.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Why Use the Template?

The [CLIF Project Template](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template) provides:

- **Consistent structure** across all CLIF projects
- **Easy federated execution** - sites know exactly where to put data and find results
- **Built-in documentation** - README, data dictionaries, analysis plans
- **Git-ready** - proper `.gitignore` for data protection

---

## Getting Started

### Create From Template

```bash
# Using GitHub CLI
gh repo create my-clif-project \
  --template Common-Longitudinal-ICU-data-Format/CLIF-Project-Template \
  --private

# Clone locally
git clone https://github.com/your-org/my-clif-project
cd my-clif-project
```

Or click "Use this template" on the [template repo page](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template).

---

## Directory Structure

```
my-clif-project/
├── README.md              # Project overview and instructions
├── ANALYSIS_PLAN.md       # Pre-registered analysis plan
├── requirements.txt       # Python dependencies
│
├── data/                  # Local data (NEVER committed)
│   ├── raw/              # Original CLIF tables
│   └── processed/        # Derived datasets
│
├── src/                   # Source code
│   ├── __init__.py
│   ├── cohort.py         # Cohort identification
│   ├── features.py       # Feature engineering
│   └── analysis.py       # Main analysis
│
├── notebooks/             # Exploratory notebooks
│   └── 01_exploration.ipynb
│
├── results/               # Output (aggregates only)
│   ├── figures/
│   └── tables/
│
├── config/                # Configuration files
│   └── config.yaml
│
└── tests/                 # Unit tests
    └── test_cohort.py
```

---

## Key Components

### README.md

Your README should include:

```markdown
# [Project Title]

## Overview
Brief description of the research question.

## Requirements
- Python 3.9+
- clifpy >= 0.5.0

## Setup
1. Clone this repository
2. Install dependencies: `pip install -r requirements.txt`
3. Place CLIF data in `data/raw/`
4. Copy `config/config.example.yaml` to `config/config.yaml`
5. Update config with your site's settings

## Running the Analysis
```bash
python -m src.analysis --config config/config.yaml
```

## Output
Results are saved to `results/`. Only aggregate statistics
are generated - no patient-level data.

## Contact
[Your name and email]
```

### config.yaml

```yaml
# Site-specific configuration
site:
  name: "UCMC"
  timezone: "America/Chicago"

# Data paths
data:
  raw_dir: "data/raw"
  processed_dir: "data/processed"
  
# CLIF tables needed
tables:
  - patient
  - hospitalization
  - vitals
  - labs
  - respiratory_support
  - medication_admin_continuous

# Cohort criteria
cohort:
  min_age: 18
  require_icu: true
  min_icu_hours: 24
  
# Analysis parameters
analysis:
  time_resolution: "1h"
  sofa_window_hours: 24
```

### .gitignore

```gitignore
# Data - NEVER commit patient data
data/
*.csv
*.parquet
*.feather

# Sensitive configs
config/config.yaml

# Python
__pycache__/
*.pyc
.venv/
*.egg-info/

# Notebooks
.ipynb_checkpoints/

# OS
.DS_Store
Thumbs.db
```

{: .warning }
**Critical:** The `data/` folder is gitignored by default. Never commit patient-level data to version control.

---

## Federated Execution Workflow

### 1. Project Lead Prepares Code

```python
# src/analysis.py
from clifpy import ClifOrchestrator
import yaml

def main(config_path: str):
    # Load config
    with open(config_path) as f:
        config = yaml.safe_load(f)
    
    # Initialize CLIF
    clif = ClifOrchestrator(data_dir=config["data"]["raw_dir"])
    clif.load_tables(config["tables"])
    
    # Build cohort
    cohort = build_cohort(clif, config["cohort"])
    
    # Run analysis
    results = run_analysis(clif, cohort, config["analysis"])
    
    # Save aggregates only
    save_results(results, "results/")
    
    print(f"✅ Analysis complete. Results saved to results/")

if __name__ == "__main__":
    main("config/config.yaml")
```

### 2. Sites Clone and Configure

```bash
# At each participating site
git clone https://github.com/clif-consortium/my-project
cd my-project
pip install -r requirements.txt

# Configure for local environment
cp config/config.example.yaml config/config.yaml
# Edit config.yaml with site-specific paths
```

### 3. Sites Run Analysis

```bash
python -m src.analysis --config config/config.yaml
```

### 4. Sites Submit Results

Only the contents of `results/` are shared with the project lead - never raw data.

---

## Testing Your Code

Include tests to catch issues before federated execution:

```python
# tests/test_cohort.py
import pytest
from src.cohort import build_cohort

def test_age_filter():
    """Cohort excludes patients under 18."""
    # Use synthetic data for testing
    result = build_cohort(synthetic_clif, {"min_age": 18})
    ages = result["age"].to_list()
    assert all(age >= 18 for age in ages)

def test_icu_requirement():
    """Cohort only includes ICU patients when required."""
    result = build_cohort(synthetic_clif, {"require_icu": True})
    # Verify all patients had ICU stay
    ...
```

Run tests before sharing code:

```bash
pytest tests/
```

---

## Documentation Standards

### ANALYSIS_PLAN.md

Pre-register your analysis plan:

```markdown
# Analysis Plan

## Primary Outcome
- 90-day mortality

## Primary Exposure
- Early vs. late prone positioning

## Inclusion Criteria
- Age ≥ 18
- ICU admission with ARDS (Berlin criteria)
- Mechanical ventilation ≥ 24 hours

## Exclusion Criteria
- Do-not-resuscitate at admission
- Transfer from outside hospital ICU

## Statistical Analysis
1. Descriptive statistics (Table 1)
2. Kaplan-Meier survival curves
3. Cox proportional hazards model
   - Adjusted for: age, sex, SOFA, P/F ratio

## Sample Size
Based on preliminary data, we expect N ≈ 5,000 across sites.
```

### Data Dictionary

Document derived variables:

```markdown
# Data Dictionary

## Cohort Variables

| Variable | Type | Description |
|----------|------|-------------|
| `hospitalization_id` | string | Unique encounter identifier |
| `icu_admit_time` | datetime | First ICU admission time |
| `intubation_time` | datetime | First mechanical ventilation |
| `prone_time` | datetime | First prone positioning (null if never) |

## Outcome Variables

| Variable | Type | Description |
|----------|------|-------------|
| `mortality_90d` | boolean | Death within 90 days of ICU admission |
| `vent_free_days_28` | integer | Ventilator-free days at day 28 |
```

---

## Example: Complete Project Setup

```bash
# Create new project from template
gh repo create clif-consortium/arf-prone-timing \
  --template Common-Longitudinal-ICU-data-Format/CLIF-Project-Template \
  --private

# Clone and setup
git clone https://github.com/clif-consortium/arf-prone-timing
cd arf-prone-timing

# Create virtual environment
python -m venv .venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows

# Install dependencies
pip install -r requirements.txt
pip install clifpy

# Start developing
code .  # Open in VS Code
```

---

## Next Steps

- Review [best practices]({{ site.baseurl }}/best-practices/) for common pitfalls
- Check out [example projects](https://github.com/Common-Longitudinal-ICU-data-Format) for inspiration
