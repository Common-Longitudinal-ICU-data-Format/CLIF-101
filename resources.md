---
layout: default
title: Resources
nav_order: 7
---

# Resources
{: .no_toc }

Essential links, repos, and references for CLIF development.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Core Repositories

| Repository | Description |
|:-----------|:------------|
| [CLIF](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF) | Data dictionary, mCIDE vocabularies, workflow docs |
| [clifpy](https://github.com/Common-Longitudinal-ICU-data-Format/clifpy) | Python package for CLIF data processing |
| [CLIF-Project-Template](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Project-Template) | Standard project structure |
| [CLIF-Lighthouse](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Lighthouse) | Web-based validation tool |
| [EHR-TO-CLIF](https://github.com/Common-Longitudinal-ICU-data-Format/EHR-TO-CLIF) | Site ETL examples |

---

## Reference Implementations

| Repository | Description |
|:-----------|:------------|
| [CLIF-MIMIC](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-MIMIC) | MIMIC-IV → CLIF ETL |
| [eICU-CLIF](https://github.com/Common-Longitudinal-ICU-data-Format/eICU-CLIF) | eICU → CLIF ETL |
| [CLIF-OMOP](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-OMOP) | OMOP ↔ CLIF conversion |

---

## Tools

| Tool | Description |
|:-----|:------------|
| [CLIF-TableOne](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-TableOne) | Generate Table 1 statistics |
| [CLIF Assistant](https://chat.openai.com) | GPT for ETL guidance (on ChatGPT) |
| [synthetic_clif](https://github.com/AartikSarma/synthetic_clif) | Generate synthetic CLIF data for testing |

---

## Claude/AI Skills

| Resource | Description |
|:---------|:------------|
| [CLIF Skills Repo](https://github.com/Common-Longitudinal-ICU-data-Format/skills) | Claude skill for CLIF development |
| [Marimo Skills](https://github.com/marimo-team/skills) | Marimo notebook skills |
| [Awesome Claude Skills](https://github.com/ComposioHQ/awesome-claude-skills) | Community Claude skills |

---

## Publications

### Primary CLIF Paper
> Rojas JC, Lyons PG, Chhikara K, et al. **A common longitudinal intensive care unit data format (CLIF) for critical illness research.** *Intensive Care Med* 51, 556–569 (2025). [doi.org/10.1007/s00134-024-07798-6](https://doi.org/10.1007/s00134-024-07798-6)

### Key Findings from Concept Paper
- External validation study: LightGBM mortality model with AUCs 0.73-0.81 across sites
- Temperature trajectory analysis: 4 phenotypes identified consistently across 9 health systems
- Cohort: 111,440 admissions from 9 systems, 39 hospitals (2020-2021)

---

## Community

| Resource | Link |
|:---------|:-----|
| CLIF Website | [clif-icu.com](https://clif-icu.com) |
| Slack | CLIF Consortium workspace |
| Epic UserWeb | [CLIF Discussion Thread](https://userweb.epic.com/Thread/136603/) |
| Weekly Calls | Thursdays 2-3 PM CT |

---

## Slack Channels

| Channel | Purpose |
|:--------|:--------|
| #general | Announcements and general discussion |
| #clif-code-ecosystem | Coding, tools, AI/Claude discussions |
| #clifpy | clifpy package development |
| #grants | Grant opportunities and proposals |
| #meetings | Call agendas and notes |

---

## mCIDE Vocabulary Files

All controlled vocabularies are available in the [CLIF skills repo](https://github.com/Common-Longitudinal-ICU-data-Format/skills/tree/main/skills/clif-icu/mCIDE):

| Table | Vocabulary File |
|:------|:----------------|
| vitals | `mCIDE/vitals/clif_vitals_categories.csv` |
| labs | `mCIDE/labs/clif_lab_categories.csv` |
| medications | `mCIDE/medication_admin_*/clif_medication_*_med_categories.csv` |
| respiratory | `mCIDE/respiratory_support/` |
| position | `mCIDE/postion/clif_position_categories.csv` |

---

## Learning Resources

### Video Tutorials
- [CLIF Overview](https://clif-icu.com) - Introduction to CLIF
- [clifpy Walkthrough](https://github.com/Common-Longitudinal-ICU-data-Format/clifpy) - Getting started with clifpy

### Example Projects
Browse the [CLIF GitHub org](https://github.com/Common-Longitudinal-ICU-data-Format) for active research projects demonstrating best practices.

---

## Contributing

Want to help improve CLIF?

- **Bug reports**: Open issues on the relevant repo
- **Code contributions**: PRs welcome! Follow existing patterns
- **Documentation**: Help improve these docs or the main CLIF documentation
- **ETL sharing**: Add your site's approach to EHR-TO-CLIF

---

*Missing a resource? Let us know in #clif-code-ecosystem!*
