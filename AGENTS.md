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

| File                                  | Purpose                                                                                                                                                                                                         |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.github/workflows/gemini-review.yml` | Reusable PR code review using Google's `gemini-3.5-flash` (via `google-github-actions/run-gemini-cli@v0`). Authenticates to Vertex AI keylessly via Workload Identity Federation — no per-repo secret required. |
| `.github/workflows/pr-title-lint.yml` | Reusable PR title lint — validates the PR title is a Conventional Commit message (needed by repos that squash-merge with `squash_merge_commit_title: PR_TITLE`, since release-please parses that title).        |

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

## How a change here actually reaches consumers

Merging to `main` does **not** release. `release-please.yml` maintains a release PR; merging
_that_ PR creates the immutable tag (`v2.3.0`) and then force-moves the major alias (`v2`) to
it. Consumers pin `@v2`, so nothing reaches them until the release PR merges.

The trap: release-please only counts **releasable** commit types. Its
`DEFAULT_CHANGELOG_SECTIONS` marks `ci`, `chore`, `docs`, `style`, `refactor`, `test` and
`build` as `hidden: true`, and hidden types are filtered out before the version bump is
computed. Since this repo prescribes the `ci` type and every change here is a workflow change,
the documented convention would silently never release — a `ci`-titled PR merges, no release PR
appears, the `v2` tag stays put, and consumers keep running the old workflow. This bit PR #37.

`release-please-config.json` therefore un-hides `ci` via `changelog-sections`. Two things to
know if you touch that config:

- The array **replaces** the defaults rather than merging with them, so every type must be
  restated. Dropping `feat`/`fix` from it would stop releasing them altogether.
- **This repo squash-merges using the PR title**, so the _PR title_ is what release-please
  parses — not the commit messages on the branch. A correctly typed commit under a
  `chore:`-titled PR still will not release.

To force a release regardless of commit types, push an empty commit whose body contains
`Release-As: x.y.z`.
