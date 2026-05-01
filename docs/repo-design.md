# Repository Design

## Goals

- Provide a central, reviewable prompt catalog for Mantle CI and GitHub Actions.
- Make prompts easy to consume from automation without requiring package
  installation.
- Keep prompt behavior reproducible by using versioned paths and commit or tag
  pinning.
- Allow future Mantle repositories to share common review, release, security,
  documentation, and maintenance prompts without copying prompt text into each
  repository.

## Non-goals

- This repository is not a runtime SDK.
- This repository does not own workflow orchestration. Actions should decide
  when and how to run a prompt suite.
- This repository should not require dependencies just to read prompts.

## Layout

```text
manifest.json
README.md
docs/
  repo-design.md
profiles/
  repositories/
    <owner>-<repo>.json
prompts/
  <domain>/
    <suite>/
      README.md
      v1/
        <prompt>.md
```

Domains should describe the workflow area, such as `pr`, `release`, `security`,
or `docs`. Suites should describe the action being performed, such as
`adversarial-review`.

## Prompt Contract

Each prompt file should be self-contained after placeholder substitution. A CI
consumer should not need to read sibling files to understand the task unless the
suite README explicitly says so.

Common placeholder names should stay stable:

- `{{REPO}}` for `owner/name`.
- `{{PR_NUMBER}}` for a GitHub pull request number.
- `{{TARGET_LABEL}}` for a human-readable target label.
- `{{REPOSITORY_PROFILE}}` for durable repo-specific facts and risk areas.
- `{{USER_FOCUS}}` for the repo-specific review emphasis.
- `{{BASE_REF}}` and `{{HEAD_REF}}` for branch names, if needed by future
  prompts.

The suite README must document all required placeholders.

## Repository Profiles

Repository profiles keep generic prompt templates from hard-coding one repo's
language, package layout, runtime, risk areas, or CI expectations.

Each profile should live at `profiles/repositories/<owner>-<repo>.json` and
include:

- `repository`: GitHub repository in `owner/name` form.
- `variables.USER_FOCUS`: a single sentence that calibrates review intensity and
  emphasis.
- `variables.REPOSITORY_PROFILE`: an array of durable facts about the repository
  and its risk model.

Profiles are deliberately plain JSON so an action can select one file per repo,
render `variables.REPOSITORY_PROFILE` as Markdown bullets, and perform string
replacement. Profile files should not contain runtime-specific values such as
the PR number.

## Versioning

Prompt versions live under `vN` directories.

- Use a new major version when prompt behavior, output contract, required tools,
  placeholders, or review order changes in a way that may affect CI results.
- Safe clarifications may stay in the current version, but consumers that need
  reproducibility should pin the repository by commit SHA or tag.
- `manifest.json` records the default active version for workflows that choose
  not to hard-code a version.

## Manifest

`manifest.json` is the machine-readable entry point. It records:

- suite id and default version
- prompt order and agent role
- file path for each prompt
- required placeholders
- profile JSON paths and profile-backed variables
- output markers used by CI to find summaries

The manifest should stay small and dependency-free so it can be parsed by shell,
Node.js, Python, or `jq` in GitHub Actions.

## First Suite: PR Adversarial Review

The first suite is `pr.adversarial-review`.

Version `v1` was extracted as-is from the active `.github/prompts` files used by
the Mantle agent scaffold. It is kept for compatibility and historical
comparison.

Version `v2` is the default. It keeps the three-stage review workflow but moves
repo-specific context into one JSON profile per repository under
`profiles/repositories/`.

The suite has three serial stages:

1. Codex initial adversarial review.
2. Claude second-pass adversarial review and cross-review of Codex findings.
3. Codex final adversarial review and disposition of Claude findings.

The prompts intentionally include concrete GitHub CLI commands and summary
markers because they are meant to run inside CI/action jobs, not as generic chat
instructions.

## Adding Future Suites

When adding a new prompt suite:

1. Create `prompts/<domain>/<suite>/README.md`.
2. Add a version directory such as `v1/`.
3. Add self-contained prompt files with stable, descriptive names.
4. Register the suite and prompts in `manifest.json`.
5. Document placeholders, profile variables, expected tools, output markers, and
   ordering.
6. Run a quick validation that every manifest path exists and every documented
   placeholder appears where expected.

When adding support for a new repository to an existing generic suite:

1. Add `profiles/repositories/<owner>-<repo>.json`.
2. Register the profile in `manifest.json`.
3. Render at least one prompt with the new profile to confirm no placeholders are
   left unresolved.
