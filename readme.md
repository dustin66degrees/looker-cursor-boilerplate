# Cursor + Looker Starter Kit

A generic, reusable boilerplate for starting **net-new LookML projects** with Cursor AI rules, skills, and a layered project structure.

Merged from:

- [purina_looker](https://github.com/<GITHUB_ORG>/purina_looker) — production layered LookML conventions
- [data-and-analytics-implementation-standards](https://github.com/<GITHUB_ORG>/data-and-analytics-implementation-standards) — LookML Cursor skills (`looker-skills-and-rules` branch)
- [gfs_looker_config](https://github.com/<GITHUB_ORG>/gfs_looker_config) — Looker instance configuration patterns (connections, roles reference)

---

## Quick Start

### 1. Bootstrap a new project

```powershell
# Copy this boilerplate into your new repo directory
cp -Recurse looker-cursor-boilerplate/* C:\path\to\<REPO_NAME>/
cd C:\path\to\<REPO_NAME>

# Initialize git and connect to GitHub
git init
git remote add origin https://github.com/<GITHUB_ORG>/<REPO_NAME>.git
```

Or clone this repo as a template and rename it.

### 2. Replace placeholders (required)

Search the entire repo for `<` and replace every placeholder before development. See the [Placeholder Checklist](#placeholder-checklist) below.

### 3. Configure MCP and credentials

```powershell
copy .cursor\mcp.json.example .cursor\mcp.json
copy .cursor\looker.env.example .cursor\looker.env
copy .cursor\github.env.example .cursor\github.env
# Optional:
copy .cursor\atlassian.env.example .cursor\atlassian.env
```

Fill in real values in the `.env` files (never commit them). Reload Cursor (**Developer: Reload Window**).

### 4. Connect Looker

1. In Looker Admin, create a Git-backed project pointing to `<GITHUB_ORG>/<REPO_NAME>`
2. Set the connection name to match `<LOOKER_CONNECTION_NAME>` in `04_models/model_name.model.lkml`
3. Enable Development Mode and pull from remote

### 5. Open in Cursor and start building

Open the project folder in Cursor. The agent automatically loads rules from `.cursor/rules/` and skills from `.cursor/skills/`.

Example prompts:

- "Extract schema for `orders` from BigQuery and generate base + standard LookML"
- "Create an explore joining orders and customers"
- "Run LAMS and fix any lint errors"

---

## What's Included

### LookML folder structure

```
00_dashboards/     LookML dashboards
01_base/           Machine-generated base views (read-only)
02_standard/       Refinement layers (*.layer.lkml)
03_logical/        Explore files (*.explore.lkml)
04_models/         Model files
05_config/         manifest, datagroups, lams.yml
06_data_tests/     LookML data tests
```

Each folder includes an instruction markdown file explaining its purpose.

### Cursor rules (`.cursor/rules/`)

| Rule | Scope | Description |
|------|-------|-------------|
| **`github_auth.mdc`** | Always on | **GitHub guardrails** — account, branch, and repo validation before push/PR |
| **`lookml_expert.mdc`** | Always on | Layered LookML generation, BigQuery extract workflow, folder conventions |
| **`lookml-best-practices.mdc`** | `*.lkml` | Use `explore_fields` sets instead of `hidden: yes` |
| **`jira_workflow.mdc`** | Optional | Jira ticket templates and GitHub linking (disabled by default) |

#### GitHub guardrails (highlight)

`github_auth.mdc` enforces:

1. **Account check** — `gh auth status` must show `<YOUR_GITHUB_HANDLE>` before any push or PR
2. **Account switch** — never use `<PERSONAL_GITHUB_HANDLE>` for client org work
3. **Remote validation** — confirm `git remote` is `<GITHUB_ORG>/<REPO_NAME>`
4. **Branch conventions** — feature branches off `<DEFAULT_BRANCH>`, no direct commits to main unless instructed
5. **Credential isolation** — GitHub MCP uses `.cursor/github.env`, not personal tokens

These rules **override** conflicting guidance from imported skills.

### Cursor skills (`.cursor/skills/`)

14 LookML skills covering project setup, views, explores, models, fields, refinements, LAMS, tests, and documentation. See `.cursor/skills/lookml-skills-README.md`.

### MCP template (`.cursor/mcp.json.example`)

Pre-configured for:

- **BigQuery** — schema extraction (`<GCP_PROJECT_ID>`)
- **Looker** — validation, queries, dev mode
- **GitHub** — PR creation and repo operations

---

## Placeholder Checklist

Replace every occurrence before starting work:

| Placeholder | Where to find the value |
|-------------|-------------------------|
| `<GITHUB_ORG>` | Your GitHub organization (e.g., company org slug) |
| `<REPO_NAME>` | New LookML repository name |
| `<PROJECT_REPO_NAME>` | Local folder / repo name (often same as `<REPO_NAME>`) |
| `<YOUR_GITHUB_HANDLE>` | Your **work** GitHub username for client repos |
| `<PERSONAL_GITHUB_HANDLE>` | Personal GitHub username to **avoid** for client work |
| `<DEFAULT_BRANCH>` | Base branch (`main` or `master`) |
| `<CLIENT_NAME>` | Client or program name (used in rules prose) |
| `<PROJECT_DISPLAY_NAME>` | Human-readable project title |
| `<GCP_PROJECT_ID>` | BigQuery GCP project ID |
| `<LOOKER_INSTANCE>` | Looker subdomain (e.g., `acme` → `acme.looker.com`) |
| `<LOOKER_CONNECTION_NAME>` | Connection name in Looker Admin / model file |
| `<model_name>` | Rename `04_models/model_name.model.lkml` accordingly |
| `<ATLASSIAN_SITE>` | Jira site slug (optional) |
| `<JIRA_PROJECT_KEY>` | Jira project key (optional) |
| `<YOUR_EMAIL>` | Atlassian account email (optional) |
| `<GITHUB_HANDLE_1>` etc. | CODEOWNERS reviewers in `.github/CODEOWNERS` |
| `<REVIEWER_GITHUB_HANDLE>` | Default PR reviewer (optional, Jira rule) |

**Quick find:** run a project-wide search for `<` in Cursor or:

```powershell
rg '<[A-Z_]+>' --glob '!*.md'
```

---

## Rule Precedence

When guidance conflicts:

1. **`.cursor/rules/github_auth.mdc`** — highest priority for git/GitHub operations
2. **`.cursor/rules/lookml_expert.mdc`** — overrides skills for schema extraction and layering
3. **`.cursor/rules/lookml-best-practices.mdc`** — field visibility on `*.lkml` edits
4. **`.cursor/skills/*`** — detailed how-to for specific LookML tasks

See `.cursor/skills/STANDARDS_MANIFEST.md` for the full merge map.

---

## Optional: Enable Jira Integration

1. Fill in `.cursor/atlassian.env` from the example file
2. Add Atlassian MCP to `.cursor/mcp.json` if needed
3. Set `alwaysApply: true` in `.cursor/rules/jira_workflow.mdc`

---

## Publishing This Boilerplate

To share as a standalone GitHub template repo:

1. Replace remaining `<GITHUB_ORG>` references in this README with your org name (or leave as documentation placeholders)
2. Push to `<GITHUB_ORG>/looker-cursor-boilerplate`
3. Enable **Template repository** in GitHub repo settings

---

## License

Internal use — adapt per your organization's standards.
