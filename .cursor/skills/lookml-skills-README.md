# LookML Cursor Skills

These skills teach the AI how to build, maintain, and document LookML projects using the layered template conventions (`01_base/`, `02_standard/`, `03_logical/`, `04_models/`, `05_config/`).

> **Project rules take precedence:** `.cursor/rules/lookml_expert.mdc`, `github_auth.mdc`, and optional `jira_workflow.mdc` override these skills when guidance conflicts. See `STANDARDS_MANIFEST.md`.

Ask in natural language — the agent will find and apply the relevant skill.

## How to Prompt for Skills

| Prompt | Skill |
|--------|-------|
| "Set up a new LookML project" | `lookml-project-setup` |
| "Generate the model documentation" | `generate-model-docs` |
| "Create a LookML view for `sales_orders`" | `lookml-view` |
| "Add an explore and join the depot view" | `lookml-explore` |
| "Structure refinements for the customer view" | `lookml-refinements` |
| "Add row-level security by region" | `lookml-access-grants` |
| "Run LAMS" | `lookml-lams` |
| "Write a LookML test for negative values" | `lookml-tests` |

## Available Skills

### Project Setup

| Skill | Purpose |
|-------|---------|
| **lookml-project-setup** | Scaffold folder structure, starter files, LAMS config, CI workflow |

### Core LookML Objects

| Skill | Purpose |
|-------|---------|
| **lookml-view** | Views — definitions, `sql_table_name`, file organization |
| **lookml-explore** | Explores, joins, access grants |
| **lookml-model** | Model files — connections, includes |
| **lookml-fields** | Dimensions, measures, filters, parameters |

### Advanced

| Skill | Purpose |
|-------|---------|
| **lookml-modeling-guidelines** | Modeling guidelines and Looker MCP usage |
| **lookml-refinements** | Includes, refinements, project structure |
| **lookml-sets** | Field sets, visibility, drill paths |
| **lookml-access-grants** | Row-level and object-level security |
| **lookml-liquid** | Liquid for dynamic SQL, HTML, links |

### Quality

| Skill | Purpose |
|-------|---------|
| **lookml-lams** | LAMS linting locally and in CI |
| **lookml-tests** | Data integrity tests |
| **generate-model-docs** | CSV data dictionary of exposed fields |

## Related Rule

`lookml-best-practices.mdc` applies when editing `*.lkml` files — prefer `explore_fields` sets over `hidden: yes` on individual fields.
