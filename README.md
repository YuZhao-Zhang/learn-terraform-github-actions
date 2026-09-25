# Automate Terraform with GitHub Actions

Companion repo for the [Automate Terraform with GitHub Actions](https://developer.hashicorp.com/terraform/tutorials/automation/github-actions) tutorial.

This fork runs an API-driven HCP Terraform workspace from GitHub Actions. Pull requests create speculative plans. Merges to `main` apply. A separate workflow posts a Claude review of the plan.

## HCP Terraform

| Setting | Value |
| --- | --- |
| Organization | `yuzhao-terraform` |
| Workspace | `learn-terraform-github-actions` |
| Workflow | API-driven |

`main.tf` uses the same `cloud` block so local Terraform CLI targets the same workspace as CI.

## Workflows

| Workflow | Trigger | What it does |
| --- | --- | --- |
| [terraform-plan.yml](.github/workflows/terraform-plan.yml) | Pull request | Uploads a speculative config, creates a plan-only run, comments add/change/destroy counts and a run link on the PR. |
| [terraform-apply.yml](.github/workflows/terraform-apply.yml) | Push to `main` | Uploads config, creates a run, applies when the run is confirmable. |
| [terraform-plan-claude.yml](.github/workflows/terraform-plan-claude.yml) | Pull request | Collects plan artifacts with [tfctl](https://releases.hashicorp.com/tfctl/) and posts a Claude review comment. |

Plan and apply jobs skip the upstream HashiCorp education repository (`hashicorp-education/learn-terraform-github-actions`).

Both plan and apply install `tfctl` 0.3.0 (checksum-verified) and query the HCP Terraform run API after create-run.

## GitHub secrets

| Secret | Used by |
| --- | --- |
| `TF_API_TOKEN` | All Terraform workflows. HCP Terraform team token with write access to the workspace. |
| `ANTHROPIC_API_KEY` | Claude plan review workflow only. |

The workspace also needs AWS credentials as environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_SESSION_TOKEN` if using temporary credentials).

## Usage

1. Open a pull request. Wait for **Terraform Plan** (and **Terraform Plan Claude Review** if `ANTHROPIC_API_KEY` is set).
2. Review the plan comment and the HCP Terraform run link.
3. Merge to `main` to trigger **Terraform Apply**.

If the workspace has auto-apply enabled, or the plan is not confirmable, the Apply step is skipped.

## Configuration

`main.tf` deploys a publicly reachable `t2.micro` EC2 instance in `us-west-2` with Apache on port 8080. Destroy the workspace resources when you are done.
