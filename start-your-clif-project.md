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

Full walkthrough: [Creator Guide](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template/blob/main/guides/creator-guide.md).

---

## 2. Buddy test it

Reach out on #general on CLIF Slack to find a buddy site. Before consortium release, **one other site** clones the finished repo and runs it end to end on **their own data**, catching site-specific assumptions, setup gaps, and any data-security issues. They record the outcome in the [Buddy Test Report](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template/blob/main/guides/buddy-test-report-template.md). See the [Buddy Testing Guide](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template/blob/main/guides/buddy-testing-guide.md) for what gets checked. 

---

## 3. Share it

Once buddy testing passes, the **PI releases** the project for the consortium run. Release the project on the #run_requests channel on Slack. Each site clones it, points `config.json` at their data, runs it, and returns the contents of `output/final_no_phi/`: aggregate results only, never patient-level data.

---

## Getting help

- **Reach out:** clif_consortium@uchicago.edu
- **GitHub Issues:** bugs in clifpy or project repos
- **Weekly CLIF Calls:** Thursdays 2-3 PM CT
