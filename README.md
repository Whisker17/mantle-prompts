# Mantle Prompts

Shared prompt catalog for Mantle CI and GitHub Actions workflows.

This repository is intentionally small: prompts are plain Markdown files,
organized by domain and version, with a machine-readable `manifest.json` for
actions that want discovery instead of hard-coded paths.

## Repository Design

- `prompts/<domain>/<suite>/<version>/` contains self-contained prompt files.
- `manifest.json` lists prompt suites, versions, prompt order, paths, and
  required placeholders.
- `docs/repo-design.md` captures the catalog rules for adding future prompt
  suites.
- Versioned prompt paths are the stable contract. Workflows should pin this repo
  by commit SHA or tag when deterministic behavior matters.

## Available Prompt Suites

### PR Adversarial Review

Path: `prompts/pr/adversarial-review/v1/`

This suite provides a three-pass daily PR review chain:

1. `codex-initial-review.md` - initial Codex adversarial review.
2. `claude-second-pass-review.md` - Claude second-pass review and cross-review
   of Codex findings.
3. `codex-final-review.md` - final Codex review and disposition of Claude
   findings.

Required placeholders:

- `__PR_NUMBER__` - GitHub pull request number.
- `__REPO__` - GitHub repository in `owner/name` form.

Example placeholder rendering:

```bash
sed \
  -e "s/__PR_NUMBER__/${PR_NUMBER}/g" \
  -e "s#__REPO__#${GITHUB_REPOSITORY}#g" \
  prompts/pr/adversarial-review/v1/codex-initial-review.md
```

## Consumption Guidance

For simple workflows, read a versioned prompt path directly. For workflows that
need to select the current default suite, read `manifest.json` and resolve
`defaultVersion`.

Prompt files are designed to be complete after placeholder substitution. Avoid
assembling partial prompt fragments in CI unless a suite explicitly documents
that behavior.
