---
layout: default
title: AI Coding Tools
nav_order: 8
---

# AI Coding Tools for CLIF
{: .no_toc }

Multiple AI-assisted coding setups are available for CLIF development. Choose the one that fits your workflow.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

The CLIF community has developed several AI coding assistant configurations. All include CLIF-specific knowledge (schema, mCIDE vocabularies, best practices) to help you write correct code faster.

---

## Comparison

| | **Official CLIF Skills** | **Kaveri's claude-setup** | **J.C.'s clif_skills_agents** |
|---|---|---|---|
| **Repo** | [Common-Longitudinal-ICU-data-Format/skills](https://github.com/Common-Longitudinal-ICU-data-Format/skills) | [kaveriC/claude-setup](https://github.com/kaveriC/claude-setup) | [RushAI-jcr/clif_skills_agents](https://github.com/RushAI-jcr/clif_skills_agents) |
| **Install** | Marketplace (`/plugin install`) | Copy files + marketplace skill | Copy files (standalone) |
| **Platforms** | Claude Code | Claude Code | Claude Code, Kilo Code (VS Code), Codex CLI |
| **Languages** | Python | Python | Python **and R** |
| **Notebooks** | — | — | marimo, Jupyter, Quarto, RMarkdown |
| **CLIF Schema** | Full mCIDE + schemas | References official skill | Full 2.1 schema embedded |
| **Session Commands** | — | `/start-clif-session`, `/end-clif-session` | `/start-session`, `/end-session` |
| **Best For** | Marketplace users | Quick Claude Code setup | Multi-platform teams, R users |

---

## Option 1: Official CLIF Skills (Marketplace)

The canonical, consortium-maintained skill. Install via Claude Code's plugin marketplace.

### Installation

```bash
# Inside Claude Code
/plugin marketplace add Common-Longitudinal-ICU-data-Format/skills
/plugin install clif-icu@clif-skills
```

### What's Included

- Full CLIF 2.1 data dictionary
- mCIDE controlled vocabularies (labs, vitals, meds, etc.)
- YAML schema definitions
- clifpy API reference
- Example scripts

{: .note }
This is the recommended approach for most users. The skill is maintained by the consortium and stays up-to-date with CLIF schema changes.

---

## Option 2: Kaveri's Claude Setup

A quick-start configuration that installs the official skill plus adds useful workflow features.

### Installation

```bash
# Clone the repo
git clone https://github.com/kaveriC/claude-setup

# Copy global config
cp claude-setup/CLAUDE.md ~/.claude/CLAUDE.md
cp claude-setup/settings.json ~/.claude/settings.json
cp claude-setup/commands/*.md ~/.claude/commands/

# Install CLIF skills plugin (inside Claude Code)
/plugin marketplace add Common-Longitudinal-ICU-data-Format/skills
/plugin install clif-icu@clif-skills
```

### What's Included

| File | Purpose |
|:-----|:--------|
| `CLAUDE.md` | Workflow rules: plan mode, verification, task tracking, lessons learned |
| `settings.json` | Extended thinking enabled, CLIF plugin, voice notification hook |
| `commands/start-clif-session.md` | `/start-clif-session` — reads progress, presents state |
| `commands/end-clif-session.md` | `/end-clif-session` — documents progress, saves lessons |

### Best For

Claude Code users who want:
- Structured session management
- Voice notifications when Claude finishes
- Built-in task tracking and lessons learned

---

## Option 3: J.C.'s clif_skills_agents

A comprehensive, multi-platform setup supporting Python, R, and multiple notebook types.

### Installation

**Claude Code (recommended):**
```bash
cd your-clif-project/
cp -r /path/to/clif_skills_agents/.claude .claude
cp /path/to/clif_skills_agents/CLAUDE.md ./CLAUDE.md
cp /path/to/clif_skills_agents/SKILL.md ./SKILL.md
```

**Kilo Code (VS Code):**
```bash
cp /path/to/clif_skills_agents/AGENTS.md ./AGENTS.md
cp -r /path/to/clif_skills_agents/.kilocode .kilocode
```

**Codex CLI:**
```bash
cp /path/to/clif_skills_agents/AGENTS.md ./AGENTS.md
```

### Auto-Detection

Just run `/start-session` — the assistant auto-detects your setup:

| Your project files | Language | Notebook | Skills loaded |
|:-------------------|:---------|:---------|:--------------|
| `.py` + marimo in pyproject.toml | Python | marimo | clif-data, python-dev, marimo, batch, figures |
| `.ipynb` only | Python | Jupyter | clif-data, python-dev, jupyter, figures |
| `.R` / `.Rmd` / renv.lock | R | RMarkdown | clif-data, r-dev, r-data-tooling, rmarkdown, figures |
| `.qmd` | R | Quarto | clif-data, r-dev, r-data-tooling, quarto, figures |

### 8 Rules That Prevent Cross-Site Bugs

The assistant enforces critical rules automatically:

1. **Filter on `_category`, never `_name`** — Categories are the CLIF controlled vocabulary
2. **Validate before analysis** — Python: `clif.validate_all()` | R: CLIF-TableOne
3. **Use ADT for ICU times, not hospitalization** — Filter `adt` where `location_category == 'icu'`
4. **Log row counts at every filter step** — CONSORT-style attrition tracking
5. **Use encounter stitching** — One ICU stay can span multiple hospitalization records
6. **Distinguish continuous vs intermittent meds** — Drips vs boluses
7. **All timestamps are UTC** — Timezone conversion is mandatory
8. **JAMA figure style** — Arial, 8pt min, no overlap, legends outside plot area

### Best For

- **R users** — Only option with R support
- **VS Code users** — Works with Kilo Code extension
- **Multi-notebook teams** — Supports marimo, Jupyter, Quarto, RMarkdown
- **Standalone setups** — No marketplace dependency

---

## Quick Config Files

### Python (clif_config.json)

```json
{
  "data_directory": "/path/to/clif/parquet/files",
  "timezone": "US/Eastern",
  "site_name": "YOUR_SITE"
}
```

### R (config/config_template.yaml)

```yaml
site: "YOUR_SITE"
file_type: "parquet"
clif_data_path: "/path/to/clif/parquet/files"
site_time_zone: "US/Eastern"
```

---

## Which Should I Use?

**Start with the Official CLIF Skills** if you:
- Use Claude Code
- Want the consortium-maintained, always-up-to-date skill
- Prefer marketplace installation

**Add Kaveri's setup** if you also want:
- Session management commands
- Voice notifications
- Task tracking built-in

**Use J.C.'s clif_skills_agents** if you:
- Need R support
- Use VS Code with Kilo Code
- Want a self-contained setup without marketplace
- Work with multiple notebook types

{: .highlight }
These are complementary! Kaveri's setup actually installs the official skill. J.C.'s is a standalone alternative for broader platform coverage.

---

## Contributing

Have improvements to share? PRs welcome to any of these repos:

- [Official CLIF Skills](https://github.com/Common-Longitudinal-ICU-data-Format/skills)
- [Kaveri's claude-setup](https://github.com/kaveriC/claude-setup)
- [J.C.'s clif_skills_agents](https://github.com/RushAI-jcr/clif_skills_agents)
