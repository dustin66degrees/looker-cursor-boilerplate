---
name: lookml-project-setup
description: Scaffold the standard <ORG_DISPLAY_NAME> LookML project folder structure. Use when the user asks to set up a new LookML project, create the project structure, initialize a LookML repo, or bootstrap from the <ORG_DISPLAY_NAME> template.
---

# LookML Project Setup

## When to Use

Use this skill when starting a **new LookML project** or **restructuring an existing project** to follow the layered LookML architecture. This creates the numbered folder layout, starter documentation, LAMS lint config, and CI workflow.

For ongoing development after setup, use the object-specific skills (`lookml-view`, `lookml-explore`, `lookml-model`, etc.).

## Before You Scaffold

Gather from the user (ask if not provided):

| Input | Example | Used for |
| :---- | :---- | :---- |
| **Project name** | `<connection_name>` | LAMS manifest name, readme, repo naming |
| **Connection name** | `<connection_name>` | Initial `.model.lkml` `connection:` parameter |
| **Model name** | `edw` | Initial model file `04_models/edw.model.lkml` |
| **Default branch for CI** | `develop` or `main` | LAMS GitHub Actions trigger branch |
| **Week start day** | `sunday`, `monday` | Model-level `week_start_day` (optional) |

Do **not** scaffold inside the `data-and-analytics-implementation-standards` repo unless the user explicitly asks. This skill targets a dedicated LookML project repository or workspace.

## Standard Folder Structure

Create these top-level directories. Each folder gets a layer instruction file so developers understand its purpose.

```
<project-root>/
├── 00_dashboards/
Γöé   └── 00_LookML_dashboards.md
├── 01_base/
Γöé   └── 01_base_layer_instructions.md
├── 02_standard/
Γöé   └── 02_standard_layer_instructions.md
├── 03_logical/
Γöé   └── 03_logical_layer_instructions.md
├── 04_models/
Γöé   ├── 04_model_file_instructions.md
Γöé   └── <model_name>.model.lkml
├── 05_config/
Γöé   ├── 05_config_instructions.md
Γöé   └── lams.yml
├── 06_data_tests/
Γöé   └── 06_data_tests_instructions.md
├── .github/
Γöé   └── workflows/
Γöé       └── lams.yml
├── .cursor/
Γöé   ├── rules/
Γöé   Γöé   └── lookml-best-practices.mdc
Γöé   └── skills/          # copy from data-and-analytics-implementation-standards
├── readme.md
└── .gitignore
```

### Layer Responsibilities

| Folder | Purpose | Edit policy |
| :----- | :------ | :---------- |
| `00_dashboards/` | LookML dashboard files (`.dashboard.lookml`) | Hand-written |
| `01_base/` | Machine-generated views from Create View From Table | **Read-only** — never edit directly |
| `02_standard/` | Refinement layers (`*.layer.lkml`), extensions, derived tables | Hand-written refinements of base views |
| `03_logical/` | One explore per file (`*.explore.lkml`) | Hand-written explore definitions |
| `04_models/` | Lightweight model files — connection, caching, includes only | Hand-written |
| `05_config/` | Shared config: datagroups, value formats, access grants, `manifest.lkml`, `lams.yml` | Hand-written |
| `06_data_tests/` | LookML data test files | Hand-written |

### File Naming Conventions

| Object | Location | Pattern |
| :----- | :------- | :------ |
| Base view | `01_base/` | `<table>.view.lkml` |
| Standard refinement | `02_standard/` | `<table>.layer.lkml` |
| Explore | `03_logical/` | `<explore_name>.explore.lkml` |
| Model | `04_models/` | `<model_name>.model.lkml` |
| Dashboard | `00_dashboards/` | `<dashboard_name>.dashboard.lookml` |
| Data test | `06_data_tests/` | `<test_name>.lkml` |

Use `snake_case` for all LookML object and file names.

## Scaffold Workflow

### 1. Create Directories

```bash
mkdir -p 00_dashboards 01_base 02_standard 03_logical 04_models 05_config 06_data_tests .github/workflows .cursor/rules
```

### 2. Create Layer Instruction Files

Write the instruction markdown files listed in the structure above. Use the content in [references/layer-instructions.md](references/layer-instructions.md).

### 3. Create Starter Model File

Create `04_models/<model_name>.model.lkml`:

```lookml
connection: "<connection_name>"
label: "<Human-readable model label>"

# week_start_day: monday  # uncomment and set if needed

include: "/03_logical/*.explore.lkml"
# include: "/00_dashboards/*.dashboard.lookml"
```

Prefer individual explore includes (`include: "/03_logical/orders.explore.lkml"`) over wildcards as the project grows. Wildcards are acceptable for an empty starter project.

### 4. Create LAMS Config

Create `05_config/lams.yml` using the starter rules in [references/lams-starter.yml](references/lams-starter.yml). Set the `name:` field to the project name.

### 5. Create GitHub Actions Workflow

Create `.github/workflows/lams.yml` using [references/lams-workflow.yml](references/lams-workflow.yml). Set the trigger branch to match the project's default branch.

### 6. Copy Cursor Skills and Rules

Copy LookML agent assets into the new project's `.cursor/` directory. Source them from:

1. **This boilerplate** — `looker-cursor-boilerplate` (recommended for net-new projects)
2. **data-and-analytics-implementation-standards** — `looker-skills-and-rules` branch

```bash
mkdir -p .cursor/rules .cursor/skills
cp -R <source>/.cursor/rules/* .cursor/rules/
cp -R <source>/.cursor/skills/* .cursor/skills/
cp <source>/.cursor/mcp.json.example .cursor/mcp.json
```

Replace `<source>` with the path to the standards repo or template repo on the user's machine.

### 7. Create Root Files

**`.gitignore`** — at minimum:

```
.vscode/
.venv/
```

**`readme.md`** — project overview with folder structure summary and LAMS local run instructions. See [references/readme-template.md](references/readme-template.md).

### 8. Initialize Git (if new repo)

Only if the user wants a new repository:

```bash
git init
git add .
git commit -m "Initial LookML project scaffold"
```

## Post-Setup Checklist

After scaffolding, tell the user:

1. **Connect the project in Looker Admin** — create the Git connection pointing to this repo and set the connection name to match the model file.
2. **Generate base views** — follow `lookml_expert.mdc`: BigQuery MCP extract → `schema_*.json` → `01_base/*.view.lkml` + `02_standard/*.layer.lkml` (or use Looker's Create View From Table if your team allows direct generation into `01_base/`).
3. **Create standard refinements** — for each base view, add a matching `02_standard/<table>.layer.lkml` with `view: +<table> { ... }`.
4. **Define explores** — add one file per explore in `03_logical/`.
5. **Run LAMS locally** — see the `lookml-lams` skill before the first push.

## Architecture Rules (Enforce During Setup)

These rules apply across all layers. Reference them when explaining the structure to the user:

1. **Never edit `01_base/`** — all view changes go through refinements in `02_standard/`.
2. **One explore per file** in `03_logical/` — keeps merge conflicts low and parsing scoped.
3. **Model files stay lightweight** — connection, caching, datagroups, and includes only; no inline explore definitions.
4. **Field visibility via sets** — use `explore_fields` at the explore level, not `hidden: yes` on individual fields (see `lookml-best-practices` rule).
5. **Cross-view calculations** — use field-only views (FOVs) without `sql_table_name`, typically in `03_logical/` or `02_standard/` depending on scope.

## Related Skills

| Next step | Skill |
| :-------- | :---- |
| Generate views from tables | `lookml-view` |
| Create explores and joins | `lookml-explore` |
| Configure model caching | `lookml-model` |
| Set up linting | `lookml-lams` |
| Generate data dictionary | `generate-model-docs` |
