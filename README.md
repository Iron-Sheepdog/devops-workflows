# 🛠️ DevOps Workflows

Centralized repository for **shared GitHub Actions workflows** used across the Iron Sheepdog organization.

Instead of duplicating CI/CD logic in every application repository, common automation lives here as [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows) that any repo in the org can call with a single `uses:` reference.

## 📋 Available Workflows

| Workflow                         | File                                                                           | Description                                                                                                                                                                                                                                                                                                                                                               |
| -------------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Deterministic Security Scans** | [`.github/workflows/security-scans.yml`](.github/workflows/security-scans.yml) | Runs [Gitleaks](https://github.com/gitleaks/gitleaks) (hardcoded-secret detection — the PR's own commits on a pull request, the full history otherwise) and [Trivy](https://github.com/aquasecurity/trivy) (dependency CVE scanning) in parallel. Hard, repeatable baselines for problems LLMs are unreliable at.                                                         |
| **Gemini Code Review**           | [`.github/workflows/gemini-review.yml`](.github/workflows/gemini-review.yml)   | Runs an automated code review on pull requests using the official [`google-github-actions/run-gemini-cli`](https://github.com/google-github-actions/run-gemini-cli) action and the [code-review extension](https://github.com/gemini-cli-extensions/code-review). Findings are posted as inline PR review comments plus a summary, with severity levels (Critical → Low). |
| **PR Title Lint**                | [`.github/workflows/pr-title-lint.yml`](.github/workflows/pr-title-lint.yml)   | Validates that the PR title is a Conventional Commit message. Needed by any repo that squash-merges with `squash_merge_commit_title: PR_TITLE`, since the PR title is what lands on `main` and what release-please parses for the next version bump.                                                                                                                      |

## 🚀 Usage — Hybrid Guardrails (recommended)

Run the deterministic security baseline first, and only spend an AI review on PRs that clear it. This is the **recommended default** for every repo: it pairs the hard, repeatable checks (Gitleaks + Trivy) with the more exploratory Gemini code review, without wasting an AI pass on a PR that fails the baseline.

> [!NOTE]
> We run the **Gitleaks CLI binary** directly rather than `gitleaks/gitleaks-action`, because that action requires a **paid license** to scan repositories owned by an organization. The Trivy action is **pinned to a commit SHA** — `trivy-action` has been the target of supply-chain attacks via mutable tags, so never pin it to `@master` or a version tag.

### Prerequisites

Both halves of this pipeline are keyless — no secrets or API keys to provision:

- **Security scans** need no authentication at all.
- **Gemini review** authenticates via Vertex AI over Workload Identity Federation (WIF). The calling workflow just needs to grant `id-token: write` (for the WIF OIDC token) plus `pull-requests: write` and `issues: write` (so the review can be posted to the pull request).

In your application repository, create `.github/workflows/pr-quality.yml` with the following content:

```yaml
name: PR Quality Guardrails

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  issues: write
  pull-requests: write
  id-token: write # Needed for Workload Identity Federation

jobs:
  security-baseline:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/security-scans.yml@v2

  gemini-review:
    needs: security-baseline # Only run the AI review if the deterministic checks pass.
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/gemini-review.yml@v2
    with:
      gemini_model: gemini-3.5-flash
```

### Excluding automated PRs

If your repository uses tools like [release-please](https://github.com/googleapis/release-please) or Dependabot, you likely want to skip the AI review on those machine-generated PRs. Add an `if` condition to the job:

```yaml
jobs:
  security-baseline:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/security-scans.yml@v2

  gemini-review:
    needs: security-baseline
    # Skip release-please version-bump PRs
    if: "!startsWith(github.head_ref, 'release-please--')"
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/gemini-review.yml@v2
```

For Dependabot, use `github.actor != 'dependabot[bot]'`. Both conditions can be combined with `&&`.

### Optional inputs — Gemini Code Review

| Input                            | Default             | Description                                                                                                                                                                                                                                                                 |
| -------------------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `additional_context`             | _(empty)_           | Extra instructions for the review (e.g. `"Focus on security vulnerabilities"`).                                                                                                                                                                                             |
| `gemini_model`                   | `gemini-3.5-flash`  | Gemini model used for the review.                                                                                                                                                                                                                                           |
| `upload_artifacts`               | `false`             | Upload the Gemini CLI's `stdout.log`, `stderr.log`, and `telemetry.log` as a workflow artifact (`gemini-output`) for diagnostics.                                                                                                                                           |
| `gemini_debug`                   | `false`             | Enable Gemini CLI debug logging and stream responses to the job log. May expose sensitive content; turn on only when diagnosing issues.                                                                                                                                     |
| `timeout_minutes`                | `7`                 | Wall-clock minutes before the review job is cancelled. Job cost scales with the PR's comment/thread history (thread-aware dedup fetches all threads, including resolved ones, before posting), so long-lived PRs with several review rounds may need more than the default. |
| `gcp_project_id`                 | `iron-sheepdog-dev` | GCP project the review authenticates against via Workload Identity Federation. Use `isd-ai-innovation` for internal OpEx/analyst repos.                                                                                                                                     |
| `gcp_location`                   | `global`            | GCP location for Vertex AI inference. `gemini-3.5-flash` requires the `global` endpoint — it isn't served on regional endpoints like `us-central1`.                                                                                                                         |
| `gcp_workload_identity_provider` | _(empty)_           | Overrides the WIF provider to impersonate. Falls back to the caller's `VERTEX_WIF_PROVIDER` repository/organization variable if omitted.                                                                                                                                    |
| `gcp_service_account`            | _(empty)_           | Overrides the service account to impersonate. Falls back to the caller's `VERTEX_REVIEW_SA` repository/organization variable if omitted.                                                                                                                                    |

Pass them via `with:` in the calling job:

```yaml
jobs:
  gemini-review:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/gemini-review.yml@v2
    with:
      additional_context: Focus on Firestore security rules and query efficiency.
```

### Optional inputs — Security Scans

| Input                   | Default         | Description                                                                  |
| ----------------------- | --------------- | ---------------------------------------------------------------------------- |
| `severity`              | `CRITICAL,HIGH` | Comma-separated Trivy severities to report.                                  |
| `trivy_exit_code`       | `"0"`           | Trivy exit code on findings. `"0"` = report-only; `"1"` = fail the build.    |
| `gitleaks_exit_code`    | `"1"`           | Gitleaks exit code on findings. `"1"` = fail the build; `"0"` = report-only. |
| `scan_ref`              | `.`             | Filesystem path Trivy scans.                                                 |
| `node_version`          | `22.x`          | Node version used for environment context.                                   |
| `run_gitleaks`          | `true`          | Toggle the Gitleaks job.                                                     |
| `run_trivy`             | `true`          | Toggle the Trivy job.                                                        |
| `gitleaks_full_history` | `false`         | Scan the whole history even on a pull request. Non-PR events always do.      |

### What Gitleaks scans

On a `pull_request` event, Gitleaks scans **only the commits the PR adds** (`merge-base..head`, merges excluded). Every other event — `schedule`, `push`, `workflow_dispatch` — scans the full history, and a PR can opt into that with `gitleaks_full_history: true`.

The reason is cost versus what the check actually tells you. On `react-app` (~30k commits reachable from GitHub refs) the full scan takes **8m24s** against a 10-minute timeout, and re-proves a fact about commits the PR never touched. Worse, a hit on an old commit blocks an author who cannot act on it: remediating a committed secret means rotating the credential and rewriting history, not editing the PR. Scoped to the PR's own commits the same scan takes well under a second, while still catching a secret that was added and then deleted within the branch — which a working-tree scan would miss.

Full-history scanning keeps its own value, since newer Gitleaks releases ship rules that catch secrets older versions missed. That belongs on a schedule against the default branch, reporting to whoever owns security, rather than on the critical path of every pull request.

> [!TIP]
> GitHub's native [secret scanning with push protection](https://docs.github.com/en/code-security/secret-scanning/introduction/about-push-protection) is the stronger control where it is available, because it rejects the push before the secret ever enters history. Treat this job as the second layer, not the only one.

### How the Gemini review works

1. A pull request is opened or updated in your repository.
2. Your workflow calls the reusable `gemini-review.yml` workflow in this repo. Authentication is handled transparently via Workload Identity Federation — nothing to configure on the caller side beyond the `id-token: write` permission.
3. The workflow checks out the code and runs the official [`google-github-actions/run-gemini-cli`](https://github.com/google-github-actions/run-gemini-cli) action with the [`code-review` extension](https://github.com/gemini-cli-extensions/code-review) (`/pr-code-review`), which reads the PR via the GitHub MCP server and reviews security, performance, reliability, maintainability, and functionality.
4. Findings are posted back to the pull request as inline review comments plus a summary, with severity levels (Critical → Low).
5. When a pull request has no issues worth raising as inline comments, the review summary ends with a final `LGTM :+1:` line as a sign-off. This is steered via the review prompt (the `code-review` extension always posts a `COMMENT`-type review, so this is a tidy comment rather than a GitHub "Approved" status), and is layered on top of any `additional_context` you pass.

### Converging across review rounds

Because every push re-runs the full review with no built-in memory of prior rounds, the workflow
bakes in standing process rules (layered on top of any `additional_context` you pass) so repeated
pushes converge on "done" instead of resurfacing an endless stream of new nits:

- **Thread-aware dedup** — before commenting, the review checks existing PR review threads
  (including resolved ones) and won't re-post a duplicate; re-raising a declined suggestion happens
  as a reply on the original thread, not a new one.
- **Single evolving summary** — the summary comment is tagged with a `<!-- gemini-review-summary -->`
  marker and updated in place on later rounds instead of stacking a new summary comment per push.
- **Severity floor for inline threads** — only `MEDIUM` and above get inline review threads; `LOW`
  observations are folded into the summary as a bullet list instead of opening a new thread.
- **Convergence rule** — if the diff since the last review only addresses prior feedback and all
  threads are resolved, the review refreshes the summary with the `LGTM :+1:` sign-off rather than
  hunting for new nits.

These are prompt-level instructions honored on a best-effort basis by the model, not
deterministically enforced — expect a few rounds of drift on any given PR. See
[issue #22](https://github.com/Iron-Sheepdog/devops-workflows/issues/22) for the full rationale and
follow-up ideas (e.g. incremental re-review of only the delta since the last reviewed commit).

## 🔀 Picking it apart (standalone workflows)

The hybrid setup above is the recommended default, but each workflow also runs fine on its own — e.g. a repo that doesn't want the AI review yet, or wants security scanning without the extra `id-token` permission.

### Code Review only

```yaml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read
  issues: write
  pull-requests: write
  id-token: write # Needed for Workload Identity Federation

jobs:
  gemini-review:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/gemini-review.yml@v2
```

That's it — every pull request will now receive an automated Gemini code review. No API key to configure. See [Excluding automated PRs](#excluding-automated-prs) and [Optional inputs — Gemini Code Review](#optional-inputs--gemini-code-review) above for the same knobs as the hybrid setup.

### Security Scans only

```yaml
name: Security Scans

on:
  pull_request:
    types: [opened, synchronize, reopened]

permissions:
  contents: read

jobs:
  security:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/security-scans.yml@v2
```

No secrets or API keys required — both tools run free. See [Optional inputs — Security Scans](#optional-inputs--security-scans) above.

### PR Title Lint only

```yaml
name: PR Title Lint

on:
  pull_request:
    types: [opened, edited, synchronize, reopened]

jobs:
  lint-title:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/pr-title-lint.yml@v2
```

Fails the check if the PR title isn't a valid Conventional Commit message (e.g. `feat(scope): summary`). Include `edited` in the trigger's `types:` — GitHub doesn't re-run status checks just because the title changed, so without it a title fixed after the fact never gets re-validated.

#### Optional inputs — PR Title Lint

| Input           | Default                                                                  | Description                                               |
| --------------- | ------------------------------------------------------------------------ | --------------------------------------------------------- |
| `allowed_types` | `build\|chore\|ci\|docs\|feat\|fix\|perf\|refactor\|revert\|style\|test` | Pipe-separated list of allowed Conventional Commit types. |

```yaml
jobs:
  lint-title:
    uses: Iron-Sheepdog/devops-workflows/.github/workflows/pr-title-lint.yml@v2
    with:
      allowed_types: build|chore|ci|docs|feat|fix|perf|refactor|revert|style|test|deps
```

## 🏷️ Versioning

This repo follows [Semantic Versioning](https://semver.org/). Each release is cut as an immutable tag (`v1.0.0`, `v1.1.0`, …), and a **moving major tag** (`v1`) always points at the latest `v1.x.x`.

| Pin       | Behaviour                                  | Use when                                                         |
| --------- | ------------------------------------------ | ---------------------------------------------------------------- |
| `@v1`     | Latest non-breaking `v1.x.x` (recommended) | Normal usage — get fixes & features, never a breaking change     |
| `@v1.2.3` | Exact release, never moves                 | You need a fully reproducible pin                                |
| `@<sha>`  | Exact commit                               | Maximum strictness / security                                    |
| `@main`   | Bleeding edge                              | Testing unreleased changes only — **not** for production callers |

Breaking changes ship under a new major tag (`v2`); the old major (`v1`) keeps working so consumers migrate on their own schedule.

Releases are automated via [release-please](https://github.com/googleapis/release-please): merging Conventional Commits to `main` opens a release PR; merging that PR cuts the tag, updates `CHANGELOG.md`, and re-points the `v1` alias.

## 🤝 Contributing

- Changes to workflows in this repository affect **all** consuming repositories pinned to that major version — review carefully before merging.
- Follow [Conventional Commits](https://www.conventionalcommits.org/) (e.g. `ci(gemini): adjust review prompt`). `feat:` → minor bump, `fix:` → patch bump, `feat!:`/`BREAKING CHANGE:` → major bump.
- Let release-please cut releases — don't create tags by hand.
