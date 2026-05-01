# Repository Design

## Goals

- Provide a central, reviewable prompt catalog for Mantle CI and GitHub Actions.
- Let downstream repositories render prompt templates with centrally managed
  repository profiles.
- Keep repository-specific context centralized in JSON profiles.
- Keep prompt behavior reproducible by letting consumers pin this repository by
  commit SHA or tag.

## Non-goals

- This repository is not a runtime SDK.
- This repository does not own workflow orchestration. Actions decide when and
  how to run a prompt suite.
- This repository does not prescribe a rendering implementation. Consumers can
  render with shell, Node.js, Python, a GitHub Action, or another workflow tool.

## Layout

```text
manifest.json
README.md
docs/
  repo-design.md
profiles/
  repositories/
    <account>/
      <repo>.json
prompts/
  <suite>/
    README.md
    <prompt>.md
```

## Prompt Contract

Templates under `prompts/<suite>/` may contain placeholders:

- `{{REPO}}` for `owner/name`.
- `{{PR_NUMBER}}` for a GitHub pull request number or a runtime expression chosen
  by the consumer.
- `{{TARGET_LABEL}}` for a human-readable target label.
- `{{REPOSITORY_PROFILE}}` for durable repo-specific facts and risk areas.
- `{{USER_FOCUS}}` for repo-specific review emphasis.

Consumers render templates with the selected JSON profile. This repository does
not commit rendered prompt artifacts.

## Repository Profiles

Repository profiles keep generic prompt templates from hard-coding one repo's
language, package layout, runtime, risk areas, or CI expectations.

Each profile lives at `profiles/repositories/<account>/<repo>.json` and includes:

- `repository`: GitHub repository in `owner/name` form.
- `variables.USER_FOCUS`: a single sentence that calibrates review intensity and
  emphasis.
- `variables.REPOSITORY_PROFILE`: an array of durable facts about the repository
  and its risk model.

Profile files should not contain runtime-specific values such as the PR number.

## Rendering

Rendering happens in the consuming repository or workflow. A consumer should:

1. Select a template from `prompts/<suite>/`.
2. Select a profile from `profiles/repositories/<account>/<repo>.json`.
3. Render `variables.REPOSITORY_PROFILE` as Markdown bullets.
4. Replace template placeholders, including runtime values such as
   `{{PR_NUMBER}}`.

## Manifest

`manifest.json` is the machine-readable entry point. It records:

- supported repository profiles
- prompt order and agent role
- template path for each prompt
- template placeholders and profile variable sources
- tools and summary markers expected by CI

The manifest should stay small and dependency-free so it can be parsed by shell,
Node.js, Python, or `jq` in GitHub Actions.

## PR Adversarial Review

The first suite is `pr.adversarial-review`. It has three serial stages:

1. Codex initial adversarial review.
2. Claude second-pass adversarial review and cross-review of Codex findings.
3. Codex final adversarial review and disposition of Claude findings.

The prompts intentionally include concrete GitHub CLI commands and summary
markers because they are meant to run inside CI/action jobs, not as generic chat
instructions.

## Adding A Repository

1. Add `profiles/repositories/<account>/<repo>.json`.
2. Register the profile in `manifest.json`.
3. Commit the profile and manifest update.
