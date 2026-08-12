# Layer Instruction File Templates

Copy each section into the corresponding folder as `<folder>_instructions.md` (or `00_LookML_dashboards.md` for dashboards).

---

## 00_dashboards/00_LookML_dashboards.md

```markdown
# LookML Dashboards

When you are ready to incorporate dashboards into the LookML layer, store the dashboard LookML in this folder.
```

---

## 01_base/01_base_layer_instructions.md

```markdown
# Base Layer Instructions

The base layer is the layer of machine-generated LookML views that makes it simple to respond to upstream data schema changes. This layer doesn't contain any hand-editing or business logic.

With an isolated base layer, adding a column is a single button click: re-run the Create View From Table LookML generator to get the latest changes. For some databases, Create View From Table will even pull in column-level descriptions into the view definition, so you'll get updated documentation for free with this approach.

## Folders

Organize your base folders to mirror how the source data resides in your database, and generate one `.view.lkml` file per raw table.

This folder hierarchy makes it clear which data powers your Looker models. This is especially helpful if you're referencing tables across different schemas and databases.
```

---

## 02_standard/02_standard_layer_instructions.md

```markdown
# Standard Layer Instructions

The standard layer is made up of hand-written LookML refinements that preserve your column changes when you update the base layer.

The standard layer makes use of refinements (notice the `+` symbol in the view definition). Refinements are similar to extends. Extends make a copy of the extended view file, but refinements make the edits in place. For low-level changes that you're comfortable applying across your entire project, refinements are a more concise way to add business logic to a view without editing the base file.

Some examples of business logic you should add to the standard layer include:

- Renaming columns to be more human-readable
- Defining primary keys or composite primary keys
- Adding descriptions and labels
- Adding measures

## Folders

Similar to the base layer, organize your standard layer folders according to your database structure, and create one `.layer.lkml` file per raw table.

The `.layer` file extension clarifies that the file contains hand-written code and not machine-generated code. Use Looker's "Create Generic LookML File" to create files with arbitrary extensions before `.lkml`.

To recap, for each table in your database there should be one machine-generated base view called `<table>.view.lkml` and one hand-written standard layer called `<table>.layer.lkml`.
```

---

## 03_logical/03_logical_layer_instructions.md

```markdown
# Logical Layer Instructions

The logical layer is the layer of Explore definitions based on standard layer views. The `include` statement will reference the standard layer so that we get the refined fields as well as the base fields.

The `.explore` extension clarifies that this file contains an Explore definition. Separating each explore into its own file solves the problem of frequent merge conflicts among developers working on different Explores simultaneously. It also allows more granular control of exposing explores for different models.

## Standard vs. Logical Layer

The line here can be a bit blurry, but generally you should put low-level, global changes like renaming columns, hiding columns, adding descriptions, and simple measures into the standard layer and more complex changes in the logical layer (e.g. cross-view calculations).

## Cross-View Calculations

Cross-view calculations are fields that use fields from more than one view file. In the traditional setup, you'd pick one of those view files and define it there with an `include` statement for the other view. But this will often throw warnings and/or errors when you re-use this view file in an Explore that doesn't have the other views joined in.

The solution is field-only views (FOVs) — view definitions without a `sql_table_name` parameter. FOVs are a great tool for defining cross-view calculations.
```

---

## 04_models/04_model_file_instructions.md

```markdown
# Model File Instructions

Now that each Explore is defined in its own file, the model file should be very lightweight. In this design, model files only include explore files plus project-level configuration like the connection, caching, and access grants.
```

---

## 05_config/05_config_instructions.md

```markdown
# Configuration Files

You can use this folder to organize and compartmentalize configuration type LookML declarations.

A single configuration file (i.e., `config.lkml`) can be used, or you can break things up into specific files for access grants, custom value format definitions, constants, datagroups, etc.

Simply include the file(s) necessary within each model file.
```

---

## 06_data_tests/06_data_tests_instructions.md

```markdown
# Data Test Files

This folder can be used to define data tests that will be used by Looker.
```
