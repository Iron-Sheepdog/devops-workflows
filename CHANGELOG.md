# Changelog

## 1.0.0 (2026-06-08)

Initial release of the reusable Gemini code review workflow.

### Features

* Reusable `gemini-review.yml` workflow built on the official `google-github-actions/run-gemini-cli` action and the `code-review` extension.
* Optional inputs: `additional_context`, `gemini_model`, `upload_artifacts`, `gemini_debug`, `max_session_turns`.
* Loads `AGENTS.md` as review context.