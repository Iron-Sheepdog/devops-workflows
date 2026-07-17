# devops-workflows — AI Context

This file provides guidance to AI coding assistants when working with this repository.

## What this repository is

Centralized repository for **shared, reusable GitHub Actions workflows** used across the
Iron-Sheepdog GitHub organization. Application repositories call these workflows via
`uses: Iron-Sheepdog/devops-workflows/.github/workflows/<file>.yml@main` instead of
duplicating CI/CD logic locally.

## Stack

- **GitHub Actions** workflow definitions only — pure YAML
- No `package.json`, no build system, no tests, no Node.js runtime
- No connection to the shared Firebase backend (Firestore, Auth, Cloud Functions)

## Workflows

| File | Purpose |
|---|---|
| `.github/workflows/gemini-review.yml` | Reusable PR code review using Google's `gemini-3.5-flash` (via `google-github-actions/run-gemini-cli@v0`). Requires the caller to pass a `GEMINI_API_KEY` secret. |
| `.github/workflows/security-scans.yml` | Reusable Gitleaks (secret detection) + Trivy (CVE) scan. No secrets required. |
| `.github/workflows/terraform-plan.yml` | Reusable Terraform plan on pull requests (one call per environment via caller matrix), keyless WIF auth, comments the filtered plan on the PR (updated in place). Read-only. |
| `.github/workflows/terraform-apply.yml` | Reusable Terraform apply — plans to a file, then applies exactly that file. No trigger of its own; the caller must gate it (e.g. `workflow_dispatch`) so applies stay human-triggered. |
| `.github/workflows/terraform-drift.yml` | Reusable scheduled drift detection — plans against live state, opens/updates/closes a GitHub issue and Slack-alerts only when the drifting-resource set changes. Report-only. |

All three `terraform-*.yml` workflows are marked `DRAFT` in their own header comments — functional and already in use by at least one consumer, but flagged by their author as not yet as hardened/reviewed as `gemini-review.yml`/`security-scans.yml`.

## Conventions

- Every workflow must use `on: workflow_call` so it is reusable from consumer repos.
- Declare all required secrets and inputs explicitly under `workflow_call` with
  `description` and `required` fields — consumers on the GitHub Free tier cannot rely
  on organization-level secrets and must pass repository secrets explicitly.
- Request the minimum `permissions` needed (e.g. `contents: read`, `pull-requests: write`).
- Pin third-party actions to a major version tag (e.g. `@v4`, `@v2`).
- Changes here affect **all** consuming repositories — review carefully before merging to `main`.
- Conventional Commits with the `ci` type (e.g. `ci(gemini): adjust review prompt`).
- See `README.md` for consumer-facing usage instructions.
