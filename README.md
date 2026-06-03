# 🛠️ DevOps Workflows

Centralized repository for **shared GitHub Actions workflows** used across the Iron Sheepdog organization.

Instead of duplicating CI/CD logic in every application repository, common automation lives here as [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) that any repo in the org can call with a single `uses:` reference.

## 📋 Available Workflows

| Workflow | File | Description |
|---|---|---|
| **Gemini Code Review** | [`.github/workflows/gemini-review.yml`](.github/workflows/gemini-review.yml) | Runs an automated, senior-engineer-grade code review on pull requests using Google's `gemini-2.5-pro` model. Findings are posted as PR comments, ordered by severity (Blocker → Nit). |

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

The calling workflow must also grant `pull-requests: write` permission so the review can be posted as a comment.

## 🚀 Usage

In your application repository, create `.github/workflows/code-review.yml` with the following content:

```yaml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  pull-requests: write

jobs:
  gemini-review:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/gemini-review.yml@main
    secrets:
      GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
```

That's it — every pull request will now receive an automated Gemini code review.

### How it works

1. A pull request is opened or updated in your repository.
2. Your workflow calls the reusable `gemini-review.yml` workflow in this repo, explicitly passing your repository's `GEMINI_API_KEY` secret.
3. The workflow checks out the code and sends the PR diff to Gemini for a rigorous review covering correctness, security, reliability, performance, maintainability, and testing.
4. The review is posted back to the pull request as a comment.

## 🤝 Contributing

- Pin callers to `@main` for the latest version, or to a tag/SHA for stability.
- Changes to workflows in this repository affect **all** consuming repositories — review carefully before merging.
- Follow [Conventional Commits](https://www.conventionalcommits.org/) (e.g. `ci(gemini): adjust review prompt`).