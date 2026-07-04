---
name: pbi-deployer
description: "Deploy a Power BI Project (PBIP) to Power BI Service / Microsoft Fabric using fabric-cicd. One-command deploy from a local PBIP folder to a target workspace. Use when user says: 'deploy dashboard', 'publish to Power BI', 'deploy to fabric', 'push PBIP', 'run deployer', 'deploy this report', 'fabric deploy', or provides a PBIP path with deployment intent."
---

# PBI Deployer

## Overview

One-command deploy of a Power BI Project (PBIP) to Power BI Service or Microsoft Fabric using `fabric-cicd`. Handles authentication, workspace resolution, and deployment in a single step.

Trigger phrases: `deploy dashboard`, `publish to Power BI`, `deploy to fabric`, `push PBIP`, `run deployer`, `deploy this report`, `fabric deploy`

---

## Prerequisites

```bash
pip install fabric-cicd
az login
```

Verify both are ready:

```bash
fabric-cicd --version
az account show
```

If `az login` is needed, prompt the user to run it in their terminal (interactive auth cannot be automated).

---

## Workflow

### 1. Collect inputs

Ask the user (in one message if not already provided):

1. **PBIP path** — full path to the `.pbip` file or project folder
2. **Target workspace name** — the Power BI / Fabric workspace to deploy to
3. **Environment** — `dev`, `test`, or `prod` (default: `dev`)

### 2. Validate PBIP structure

Confirm:
- `.pbip` file exists
- `.Report/` subfolder exists
- `.SemanticModel/` subfolder exists

If any are missing, stop and report the issue.

### 3. Pre-deployment check

```bash
fabric-cicd validate --source "<pbip_folder>" --workspace "<workspace_name>"
```

Show the validation output. If validation fails, stop and show the error — do not proceed to deploy.

### 4. Deploy

```bash
fabric-cicd deploy \
  --source "<pbip_folder>" \
  --workspace "<workspace_name>" \
  --environment "<environment>"
```

Show the full output in real time. Do not suppress errors.

### 5. Verify deployment

After a successful deploy:

```bash
fabric-cicd status --workspace "<workspace_name>" --item "<report_name>"
```

Report the deployment status to the user.

### 6. Report to user

```
✅ Deployment complete

Dashboard: <report_name>
Workspace: <workspace_name>
Environment: <environment>
URL: https://app.powerbi.com/groups/<workspace_id>/reports/<report_id>

Next: Open in Power BI Service and trigger a dataset refresh.
```

---

## Error Handling

| Error | Action |
|---|---|
| `az login` required | Ask user to run `az login` in their terminal first |
| Workspace not found | Check workspace name spelling and user permissions |
| Validation failure | Show full `fabric-cicd validate` output |
| Deploy failure | Show full error traceback — never suppress |
| Timeout | Suggest re-running; fabric-cicd deployments are idempotent |

---

## Rules

- **Never skip validation** — always run `fabric-cicd validate` before `deploy`.
- **Show full output** — never suppress errors or summarise tracebacks.
- **Idempotent** — re-running deploy on the same PBIP is safe; it updates the existing report in the workspace.
- **Environment matters** — `prod` deployments should be confirmed by the user before proceeding (ask: *"You're deploying to prod — confirm?"*).
- **Auth is interactive** — `az login` must be run by the user in their terminal; Claude Code cannot perform interactive browser auth.
