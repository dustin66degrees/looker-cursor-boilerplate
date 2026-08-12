---
name: lookml-modeling-guidelines
description: Guidelines for LookML modeling and effective Looker MCP tool use.
---

# LookML Modeling Guidelines

> **Project rule override:** When this skill conflicts with `.cursor/rules/lookml_expert.mdc`, follow **lookml_expert** first. For <CLIENT_NAME> work that means: do **not** use the Looker MCP or web search to blindly discover fields/schemas; use BigQuery MCP + local `schema_*.json` extraction, then generate layered LookML in-repo. Use Looker MCP only when the user explicitly asks for validation, query checks, or instance-specific debugging—not for open-ended discovery.

This guide provides instructions on how to effectively use the Looker MCP
tools to assist with LookML modeling tasks. It synthesizes best practices
for LookML development with effective tool usage.

## LookML File Types

Understand the purpose of different file types in a LookML project:

*   **Project Manifest (`manifest.lkml`)**: Global configuration for the
    project, including imports and constants.
*   **Model Files (`.model.lkml`)**: Defines database connections and Explores
    (how views are joined).
*   **View Files (`.view.lkml`)**: Blueprints for data, defining dimensions,
    measures, and derived tables.
*   **Dashboard Files (`.dashboard.lookml`)**: LookML-defined dashboards
    (layout, visualizations, filters).
*   **Specialized Files**: Refinement files (for overriding/patching), Data
    Test files (automated logic checks), and Document files (`.md`).

## 1. Discovery & Exploration

*   **Project Mapping**: Use `get_projects`, `get_models`, and `get_explores` to
    understand the project structure.
*   **Field Discovery**: Use `get_dimensions` and `get_measures` to see what is
    exposed in an explore. Prefer this over reading all view files.
*   **Schema Discovery**: Use `get_connections`, `get_connection_databases`,
    `get_connection_schemas`, `get_connection_tables`, and
    `get_connection_table_columns` to inspect the connected database.
*   **Data Discovery**: Use `query` to query an explore. Note that the fields
    passed to this tool are the fully qualified names as exposed by Looker,
    which may differ from the raw field name in the view file. Always use the
    Field Discovery tools to understand the available fields before running a
    query.

## 2. Verification & Testing

Always verify that your LookML is valid and generates the expected SQL or
results.

*   **Syntactic Correctness:** All code must be syntactically perfect, with
    balanced braces and correct parameters. `sql` parameters must match the
    specific database dialect (e.g., BigQuery, Snowflake). Always ensure strict
    adherence to LookML syntax, especially when defining blocks and using
    correct indentation.
*   **Running Queries**: Use `query` to verify results.
*   **SQL Verification**: Use `query_sql` to inspect the SQL Looker
    generates but without executing the query against the database. Do not
    use this tool to generate SQL to manually execute, prefer `query` to do
    that.
*   **Validation**: Use `validate_project` frequently during development.
*   **Testing**: Use `get_lookml_tests` and `run_lookml_tests` to execute tests.

## 3. Creating New Views

Use `create_view_from_table` to generate boilerplate LookML views directly
from the database schema. Always use this tool to generate views from tables
and then edit the generated view file.

> **Project rule override:** Do not use `create_view_from_table` for base views in this project. Follow `lookml_expert.mdc`: extract schema via BigQuery MCP → save `schema_*.json` locally → generate `01_base/*.view.lkml` and `02_standard/*.layer.lkml` in this repo.

### Naming & Uniqueness Requirements

*   **Model:** Instance-wide uniqueness required (prevents URL collisions and
    instance errors).
*   **View:** Project-wide uniqueness required (acts as namespace for fields).
*   **Explore:** Model-wide uniqueness required (query starting point).
*   **Field:** View-wide uniqueness required (dimension/measure names must be
    unique within the view).

## 4. Feedback Loop (Validation)

After making any changes to LookML files:

1.  Run `validate_project` to check for syntax and reference errors.
2.  If errors are found, fix them and repeat step 1.
3.  Run `run_lookml_tests` to ensure all data tests still pass.
4.  **DO NOT** consider the task complete until `validate_project`
    returns no errors and tests pass.

---

## 5. Best Practices

### A. Models

*   **Includes**: Be specific with includes to prevent performance bloat and
    namespace collisions. Avoid broad wildcards like `/**/*.view` if possible.

### B. Explores

*   **Joins**: Always specify the `relationship` parameter explicitly (e.g.,
    `many_to_one`). This is critical for Looker to generate correct SQL and
    avoid fanouts.

### C. Views

*   **Primary Key**: Every view representing a table should have a primary key
    defined. It must be the first dimension and have `primary_key: yes`. This is
    essential for symmetric aggregates.
*   **Field References**: Measures should reference dimensions (e.g.,
    `${dimension_name}`), not table columns directly (e.g.,
    `${TABLE}.column_name`). This ensures a single source of truth.

### Constraints & Guardrails

* **Application Scope:** All guidelines and best practices in this file apply
  ONLY to new code added to fulfill the user prompt. Do not refactor or modify
  existing code to comply with these guidelines unless explicitly requested or
  necessary for the new code to function. Keep existing code as is.
* **Zero-Error Policy:** Strictly forbidden from submitting code that fails the
  LookML Validator.
* **Data Minimization:** Access only the data and schema necessary for the
  current task.
* **Validation**: Keep the scope of validation to your changes only. Do not fix
  existing validation errors.
