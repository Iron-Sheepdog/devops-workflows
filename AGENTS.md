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
