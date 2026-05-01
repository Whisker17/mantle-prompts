# Mantle Prompts

Shared prompt catalog for Mantle CI and GitHub Actions workflows.

This repository is intentionally small: prompts are plain Markdown files,
organized by domain and version, with a machine-readable `manifest.json` for
actions that want discovery instead of hard-coded paths.

## Repository Design

- `prompts/<domain>/<suite>/<version>/` contains self-contained prompt files.
- `profiles/repositories/<owner>-<repo>.json` contains repo-specific prompt
  variables, such as `{{REPOSITORY_PROFILE}}` and `{{USER_FOCUS}}`.
- `manifest.json` lists prompt suites, versions, prompt order, paths, and
  required placeholders and profile sources.
- `docs/repo-design.md` captures the catalog rules for adding future prompt
  suites.
- Versioned prompt paths are the stable contract. Workflows should pin this repo
  by commit SHA or tag when deterministic behavior matters.

## Available Prompt Suites

### PR Adversarial Review

Default path: `prompts/pr/adversarial-review/v2/`

This suite provides a three-pass daily PR review chain:

1. `codex-initial-review.md` - initial Codex adversarial review.
2. `claude-second-pass-review.md` - Claude second-pass review and cross-review
   of Codex findings.
3. `codex-final-review.md` - final Codex review and disposition of Claude
   findings.

Runtime placeholders:

- `{{PR_NUMBER}}` - GitHub pull request number.
- `{{REPO}}` - GitHub repository in `owner/name` form.
- `{{TARGET_LABEL}}` - human-readable target label, such as
  `PR #123 in owner/repo`.

Profile-backed placeholders:

- `{{REPOSITORY_PROFILE}}` - durable repo context.
- `{{USER_FOCUS}}` - concise review emphasis for the repo.

Initial repository profiles:

- [Whisker17/mantle-agent-scaffold](profiles/repositories/whisker17-mantle-agent-scaffold.json)
- [mantle-xyz/reth](profiles/repositories/mantle-xyz-reth.json)
- [mantlenetworkio/mantle-v2](profiles/repositories/mantlenetworkio-mantle-v2.json)
- [mantlenetworkio/op-geth](profiles/repositories/mantlenetworkio-op-geth.json)

Example rendering:

```bash
PR_NUMBER=123 GITHUB_REPOSITORY=Whisker17/mantle-agent-scaffold python3 - <<'PY'
from pathlib import Path
import json
import os

profile = json.loads(Path("profiles/repositories/whisker17-mantle-agent-scaffold.json").read_text())
repository_profile = "\n".join(f"- {item}" for item in profile["variables"]["REPOSITORY_PROFILE"])
template = Path("prompts/pr/adversarial-review/v2/codex-initial-review.md").read_text()
rendered = (template
  .replace("{{PR_NUMBER}}", os.environ["PR_NUMBER"])
  .replace("{{REPO}}", os.environ["GITHUB_REPOSITORY"])
  .replace("{{TARGET_LABEL}}", f"PR #{os.environ['PR_NUMBER']} in {os.environ['GITHUB_REPOSITORY']}")
  .replace("{{REPOSITORY_PROFILE}}", repository_profile)
  .replace("{{USER_FOCUS}}", profile["variables"]["USER_FOCUS"]))
print(rendered)
PY
```

`v1` is preserved as the original Mantle agent scaffold extraction and still uses
`__PR_NUMBER__` and `__REPO__`.

## Consumption Guidance

For simple workflows, read a versioned prompt path directly. For workflows that
need to select the current default suite, read `manifest.json` and resolve
`defaultVersion`.

Prompt files are designed to be complete after placeholder substitution. The
default v2 suite intentionally separates generic review behavior from
repo-specific profile content so other repositories can reuse the same review
chain by adding a profile directory.
