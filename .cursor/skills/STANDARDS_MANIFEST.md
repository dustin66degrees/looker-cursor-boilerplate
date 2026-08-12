# LookML Standards Manifest

This starter kit merges rules and skills from:

- [data-and-analytics-implementation-standards](https://github.com/<GITHUB_ORG>/data-and-analytics-implementation-standards) (`looker-skills-and-rules` branch)
- Production LookML project conventions (layered `01_base` / `02_standard` architecture)
- Authoritative local guardrails in `.cursor/rules/` (these **override** imported skills on conflict)

## Rule precedence

When guidance conflicts, **`.cursor/rules/` wins**:

| Rule | Purpose |
|------|---------|
| `github_auth.mdc` | GitHub account, branch, and repo validation |
| `lookml_expert.mdc` | Layering, BigQuery extract workflow, folder map |
| `lookml-best-practices.mdc` | Use `explore_fields` sets instead of `hidden: yes` |
| `jira_workflow.mdc` | Optional Jira integration (disabled by default) |

## Included skills

See `lookml-skills-README.md` for prompt examples.

- `lookml-project-setup` — scaffold folder structure, LAMS, CI
- `lookml-view`, `lookml-explore`, `lookml-model`, `lookml-fields`
- `lookml-refinements`, `lookml-sets`, `lookml-access-grants`, `lookml-liquid`
- `lookml-modeling-guidelines`, `lookml-lams`, `lookml-tests`, `generate-model-docs`

## Conventions enforced by project rules

- Standard refinements: `02_standard/<view_name>.layer.lkml`
- Models: `04_models/<model_name>.model.lkml`
- Base views: `01_base/<table_name>.view.lkml` (machine-generated; do not hand-edit)
