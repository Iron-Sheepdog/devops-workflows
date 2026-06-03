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
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/gemini-review.yml@main
    secrets:
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
```

That's it — every pull request will now receive an automated Gemini code review.

### Optional inputs

| Input | Default | Description |
|---|---|---|
| `additional_context` | _(empty)_ | Extra instructions for the review (e.g. `"Focus on security vulnerabilities"`). |
| `gemini_model` | `gemini-2.5-pro` | Gemini model used for the review. |

Pass them via `with:` in the calling job:

```yaml
jobs:
  gemini-review:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/gemini-review.yml@main
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

## 🤝 Contributing

- Pin callers to `@main` for the latest version, or to a tag/SHA for stability.
- Changes to workflows in this repository affect **all** consuming repositories — review carefully before merging.
- Follow [Conventional Commits](https://www.conventionalcommits.org/) (e.g. `ci(gemini): adjust review prompt`).