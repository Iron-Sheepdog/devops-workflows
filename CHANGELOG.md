# Changelog

## [2.2.1](https://github.com/Iron-Sheepdog/devops-workflows/compare/v2.2.0...v2.2.1) (2026-08-05)


### Bug Fixes

* **release-please:** make ci-type commits releasable ([#38](https://github.com/Iron-Sheepdog/devops-workflows/issues/38)) ([607f6f9](https://github.com/Iron-Sheepdog/devops-workflows/commit/607f6f9187475a114014d8e919537011aa423253))


### Continuous Integration

* **security-scans:** scan only the PR's commits with Gitleaks ([#37](https://github.com/Iron-Sheepdog/devops-workflows/issues/37)) ([c086279](https://github.com/Iron-Sheepdog/devops-workflows/commit/c0862799ff328dea3cf689225bb238e14bcf35fd))

## [2.2.0](https://github.com/Iron-Sheepdog/devops-workflows/compare/v2.1.0...v2.2.0) (2026-07-28)


### Features

* **gemini-review:** expose timeout_minutes as a configurable input ([#35](https://github.com/Iron-Sheepdog/devops-workflows/issues/35)) ([95b2add](https://github.com/Iron-Sheepdog/devops-workflows/commit/95b2adde930d84b5eeb8637244fdadc91de62fe1))

## [2.1.0](https://github.com/Iron-Sheepdog/devops-workflows/compare/v2.0.1...v2.1.0) (2026-07-23)


### Features

* **ci:** add reusable PR title lint workflow ([#33](https://github.com/Iron-Sheepdog/devops-workflows/issues/33)) ([269e15c](https://github.com/Iron-Sheepdog/devops-workflows/commit/269e15c7b5e19839509cae401e096734ecdf1c36))

## [2.0.1](https://github.com/Iron-Sheepdog/devops-workflows/compare/v2.0.0...v2.0.1) (2026-07-21)


### Bug Fixes

* **gemini-review:** bake in a shared WIF default so Free-tier private repos work ([#31](https://github.com/Iron-Sheepdog/devops-workflows/issues/31)) ([d51dc19](https://github.com/Iron-Sheepdog/devops-workflows/commit/d51dc19b193c8cdad17b4044de97e2d46496a6fb))

## [2.0.0](https://github.com/Iron-Sheepdog/devops-workflows/compare/v1.3.0...v2.0.0) (2026-07-20)


### ⚠ BREAKING CHANGES

* **gemini:** the workflow_call GEMINI_API_KEY secret no longer exists. Callers still passing `secrets: GEMINI_API_KEY: ...` will fail with "secret ... is not defined in the referenced workflow" and must remove that block (no replacement secret needed - auth is automatic via WIF once infrastructure publishes the org-level GCP variables).

### Continuous Integration

* **gemini:** switch reusable review workflow to Vertex AI via WIF ([#27](https://github.com/Iron-Sheepdog/devops-workflows/issues/27)) ([d4914aa](https://github.com/Iron-Sheepdog/devops-workflows/commit/d4914aa5bd908d4098d35809022b7779526d07a0))

## [1.3.0](https://github.com/Iron-Sheepdog/devops-workflows/compare/v1.2.0...v1.3.0) (2026-06-18)


### Features

* **gemini-review:** increase max_session_turns default from 25 to 50 ([#16](https://github.com/Iron-Sheepdog/devops-workflows/issues/16)) ([6a85ab2](https://github.com/Iron-Sheepdog/devops-workflows/commit/6a85ab2a62aa5a5f367b6728b473280bc83fce7e))

## [1.2.0](https://github.com/Iron-Sheepdog/devops-workflows/compare/v1.1.0...v1.2.0) (2026-06-12)


### Features

* **ci:** add deterministic security scans (Gitleaks + Trivy) ([#11](https://github.com/Iron-Sheepdog/devops-workflows/issues/11)) ([3d120c3](https://github.com/Iron-Sheepdog/devops-workflows/commit/3d120c38689702c00526bf2b037e2a8da1e2cd31)), closes [#8](https://github.com/Iron-Sheepdog/devops-workflows/issues/8)

## [1.1.0](https://github.com/Iron-Sheepdog/devops-workflows/compare/v1.0.0...v1.1.0) (2026-06-10)


### Features

* **gemini-review:** sign off clean PRs with "LGTM :+1:" ([#12](https://github.com/Iron-Sheepdog/devops-workflows/issues/12)) ([0737771](https://github.com/Iron-Sheepdog/devops-workflows/commit/073777199273b8e5a3667ae0500d6180b3c4ac7d))

## 1.0.0 (2026-06-08)

Initial release of the reusable Gemini code review workflow.

### Features

* Reusable `gemini-review.yml` workflow built on the official `google-github-actions/run-gemini-cli` action and the `code-review` extension.
* Optional inputs: `additional_context`, `gemini_model`, `upload_artifacts`, `gemini_debug`, `max_session_turns`.
* Loads `AGENTS.md` as review context.
