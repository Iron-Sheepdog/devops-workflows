# 🛠️ DevOps Workflows

Centralized repository for **shared GitHub Actions workflows** used across the Iron Sheepdog organization.

Instead of duplicating CI/CD logic in every application repository, common automation lives here as [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) that any repo in the org can call with a single `uses:` reference.

## 📋 Available Workflows

| Workflow | File | Description |
|---|---|---|
| **Gemini Code Review** | [`.github/workflows/gemini-review.yml`](.github/workflows/gemini-review.yml) | Runs an automated code review on pull requests using the official [`google-github-actions/run-gemini-cli`](https://github.com/google-github-actions/run-gemini-cli) action and the [code-review extension](https://github.com/gemini-cli-extensions/code-review). Findings are posted as inline PR review comments plus a summary, with severity levels (Critical → Low). |

## ✅ Prerequisites

> [!IMPORTANT]
> Due to **GitHub Free tier limitations**, organization-level secrets cannot be shared with private repositories. Each target repository must therefore configure its **own** `GEMINI_API_KEY`.

In the repository that will call this workflow:

1. Obtain a Gemini API key from [Google AI Studio](https://aistudio.google.com/apikey).
2. Navigate to **Settings → Secrets and variables → Actions**.
3. Click **New repository secret** and create:

   | Name | Value |
   |---|---|
   | `GEMINI_API_KEY` | Your Gemini API key |

The calling workflow must also grant `pull-requests: write` and `issues: write` permissions so the review can be posted to the pull request.

## 🚀 Usage

In your application repository, create `.github/workflows/code-review.yml` with the following content:

```yaml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  issues: write
  pull-requests: write

jobs:
  gemini-review:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/gemini-review.yml@v1
    secrets:
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
```

That's it — every pull request will now receive an automated Gemini code review.

### Optional inputs

| Input | Default | Description |
|---|---|---|
| `additional_context` | _(empty)_ | Extra instructions for the review (e.g. `"Focus on security vulnerabilities"`). |
| `gemini_model` | `gemini-3.5-flash` | Gemini model used for the review. |
| `upload_artifacts` | `false` | Upload the Gemini CLI's `stdout.log`, `stderr.log`, and `telemetry.log` as a workflow artifact (`gemini-output`) for diagnostics. |
| `gemini_debug` | `false` | Enable Gemini CLI debug logging and stream responses to the job log. May expose sensitive content; turn on only when diagnosing issues. |

Pass them via `with:` in the calling job:

```yaml
jobs:
  gemini-review:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/gemini-review.yml@v1
    with:
      additional_context: Focus on Firestore security rules and query efficiency.
    secrets:
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
```

### How it works

1. A pull request is opened or updated in your repository.
2. Your workflow calls the reusable `gemini-review.yml` workflow in this repo, explicitly passing your repository's `GEMINI_API_KEY` secret.
3. The workflow checks out the code and runs the official [`google-github-actions/run-gemini-cli`](https://github.com/google-github-actions/run-gemini-cli) action with the [`code-review` extension](https://github.com/gemini-cli-extensions/code-review) (`/pr-code-review`), which reads the PR via the GitHub MCP server and reviews security, performance, reliability, maintainability, and functionality.
4. Findings are posted back to the pull request as inline review comments plus a summary, with severity levels (Critical → Low).
5. When a pull request has no issues worth raising as inline comments, the review summary ends with a final `LGTM :+1:` line as a sign-off. This is steered via the review prompt (the `code-review` extension always posts a `COMMENT`-type review, so this is a tidy comment rather than a GitHub "Approved" status), and is layered on top of any `additional_context` you pass.

## 🏷️ Versioning

This repo follows [Semantic Versioning](https://semver.org/). Each release is cut as an immutable tag (`v1.0.0`, `v1.1.0`, …), and a **moving major tag** (`v1`) always points at the latest `v1.x.x`.

| Pin | Behaviour | Use when |
|---|---|---|
| `@v1` | Latest non-breaking `v1.x.x` (recommended) | Normal usage — get fixes & features, never a breaking change |
| `@v1.2.3` | Exact release, never moves | You need a fully reproducible pin |
| `@<sha>` | Exact commit | Maximum strictness / security |
| `@main` | Bleeding edge | Testing unreleased changes only — **not** for production callers |

Breaking changes ship under a new major tag (`v2`); the old major (`v1`) keeps working so consumers migrate on their own schedule.

Releases are automated via [release-please](https://github.com/googleapis/release-please): merging Conventional Commits to `main` opens a release PR; merging that PR cuts the tag, updates `CHANGELOG.md`, and re-points the `v1` alias.

## 🤝 Contributing

- Changes to workflows in this repository affect **all** consuming repositories pinned to that major version — review carefully before merging.
- Follow [Conventional Commits](https://www.conventionalcommits.org/) (e.g. `ci(gemini): adjust review prompt`). `feat:` → minor bump, `fix:` → patch bump, `feat!:`/`BREAKING CHANGE:` → major bump.
- Let release-please cut releases — don't create tags by hand.