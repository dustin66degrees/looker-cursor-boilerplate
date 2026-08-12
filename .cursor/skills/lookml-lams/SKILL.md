---
name: lookml-lams
description: Run and configure LAMS (Look At Me Sideways) linting locally and in CI for a LookML project. Use when the user asks to run LAMS, add or update LookML lint rules, or troubleshoot CI lint failures.
---

# LookML LAMS Linting

## Project Setup

- **LAMS package:** `@looker/look-at-me-sideways@5` ([docs](https://looker-open-source.github.io/look-at-me-sideways/))
- **Rule configuration:** `05_config/lams.yml` (passed via `--manifest`; keeps `manifest.lkml` focused on Looker project settings)
- **CI workflow:** `.github/workflows/lams.yml` (runs on push/PR to `develop`)

## Running Locally

```bash
npm install -g @looker/look-at-me-sideways@5 js-yaml
cd /path/to/your-lookml-project
lams --reporting=no --manifest=05_config/lams.yml --output=lines
```

Use `--verbose` for additional detail. See [customizing LAMS](https://looker-open-source.github.io/look-at-me-sideways/customizing-lams) for custom expression rules.

## Current Starter Rules

| Rule | Type | Purpose |
|------|------|---------|
| `explore_join_relationship` | Custom | Every explore join must declare `relationship:` (project standard per `lookml-explore` skill) |
| `E1` | Built-in | Join `sql_on` must use `${view.field}` substitution |
| `E6` | Built-in | `foreign_key` joins must not be `*-to-many` |
| `F2` | Built-in | Fields must not use `view_label` (use explore join `view_label` instead) |

## Adding Rules Later

1. Edit `05_config/lams.yml` and add or uncomment a rule entry.
2. Run LAMS locally and fix any new violations before pushing.
3. For legacy violations, use incremental adoption:
   ```bash
   lams --reporting=no --manifest=05_config/lams.yml --output=add-exemptions
   ```
   Commit the resulting `lams-exemptions.ndjson` ([docs](https://looker-open-source.github.io/look-at-me-sideways/github-action.html)).

## Rule Exemptions (Inline)

For intentional one-off exceptions, use inline exemptions in LookML:

```lookml
# LAMS exempt: E1 {why: "document reason here"}
```

## Conventions This Linter Supports

- **One explore per file** in `03_logical/` (convention; not yet a LAMS rule)
- **Explicit join relationships** on all explores
- **Refinement layers** in `02_standard/` (do not edit `01_base/` directly)
- **Field exposure** via `explore_fields` sets (documented by `generate-model-docs` skill)

## Future Rule Candidates

Enable when the team is ready to address existing violations:

- `K7` — table-backed views declare exactly one `primary_key`
- `F1` — no cross-view references in field SQL
- `F3` — `type: count` measures specify filters
- `F4` — non-hidden fields have descriptions
- `W1` — block indentation
