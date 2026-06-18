# Changelog

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
