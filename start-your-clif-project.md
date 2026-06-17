---
layout: default
title: Start Your CLIF Project
nav_order: 2
---

# Start Your CLIF Project
{: .no_toc }

Everything you need to turn your idea into a reproducible, consortium-ready CLIF project.
{: .fs-6 .fw-300 }

A CLIF project comes together in three phases:

- **Code it:** the creator builds the analysis from the template.
- **Buddy test it:** one other site runs it end to end and signs off.
- **Share it:** the PI releases it, every site runs it and returns **aggregate results only**.

---

## 1. Code it

**Start from the template.** Click **"Use this template"** on the [CLIF Project Template](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template), or:

```bash
gh repo create my-clif-project \
  --template Common-Longitudinal-ICU-data-Format/CLIF-Project-Template --private
```

You get a ready structure:

```
my-clif-project/
├── README.md                 # the run-it-yourself guide each site reads
├── config/config_template.json   # site settings (copy to config.json)
├── code/templates/{Python,R}/    # numbered pipeline scripts
├── output/
│   ├── final_no_phi/         # aggregate, shareable results
│   └── intermediate_phi/     # patient-level working data, NEVER shared (gitignored)
├── utils/                    # config loaders for Python & R
└── guides/                   # creator guide, primer, buddy-testing guide
```

**Configure your site.** Copy `config_template.json` to `config.json` (gitignored) and set `site_name`, `tables_path`, and `file_type`. Keep *all* site-specific values here, and never hardcode paths in scripts.

**Set up the environment.** Python uses [uv](https://docs.astral.sh/uv/) (`uv sync`); R uses [renv](https://rstudio.github.io/renv/) (`00_renv_restore.R`). Commit the lockfile so every site reproduces the same packages.

**Write the pipeline.** Organize your code so another site can run it in order: typically cohort identification, then quality checks, then analysis. This is a suggestion, not a rule. The [Project Primer](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template/blob/main/guides/primer.md) covers cohort building and optimization tips.

{: .warning }
**Only aggregate results leave your site.** No `patient_id` or row-level records, every statistic at **n ≥ 10**, no raw `.csv`/`.parquet`. Shareable results go in `output/final_no_phi/`; patient-level working files stay in `output/intermediate_phi/` (gitignored).

Full walkthrough: [Creator Guide](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template/blob/main/guides/creator-guide.md).

---

## 2. Buddy test it

Reach out on #general on CLIF Slack to find a buddy site. Before consortium release, **one other site** clones the finished repo and runs it end to end on **their own data**, catching site-specific assumptions, setup gaps, and any data-security issues. They record the outcome in the [Buddy Test Report](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template/blob/main/guides/buddy-test-report-template.md). See the [Buddy Testing Guide](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template/blob/main/guides/buddy-testing-guide.md) for what gets checked. 

---

## 3. Share it

Once buddy testing passes, the **PI releases** the project for the consortium run. Release the project on the #run_requests channel on Slack. Each site clones it, points `config.json` at their data, runs it, and returns the contents of `output/final_no_phi/`: aggregate results only, never patient-level data.

---

## Getting help

- **#clif-code-ecosystem:** coding questions
- **#clifpy:** clifpy-specific issues
- **GitHub Issues:** bugs in clifpy or project repos
- **Weekly CLIF Calls:** Thursdays 2-3 PM CT
