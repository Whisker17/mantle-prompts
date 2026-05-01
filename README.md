# Mantle Prompts

Shared prompt catalog for Mantle CI and GitHub Actions workflows.

The repository keeps prompt templates and repository profiles together.
Downstream repositories render the templates with the profile they need.

## Repository Design

- `prompts/<suite>/` contains generic prompt templates.
- `profiles/repositories/<account>/<repo>.json` contains repo-specific prompt
  variables, such as `{{REPOSITORY_PROFILE}}` and `{{USER_FOCUS}}`.
- `manifest.json` lists prompt suites, prompt order, profile paths, template
  variables, output markers, and required tools.
- `docs/repo-design.md` captures the catalog rules for adding future prompt
  suites and repository profiles.

## PR Adversarial Review

Template path: `prompts/adversarial-review/`

This suite provides a three-pass daily PR review chain:

1. `codex-initial-review.md` - initial Codex adversarial review.
2. `claude-second-pass-review.md` - Claude second-pass review and cross-review
   of Codex findings.
3. `codex-final-review.md` - final Codex review and disposition of Claude
   findings.

Templates require these variables:

- `{{PR_NUMBER}}` - GitHub pull request number or runtime expression chosen by
  the consumer.
- `{{REPO}}` - GitHub repository in `owner/name` form.
- `{{TARGET_LABEL}}` - human-readable target label.
- `{{REPOSITORY_PROFILE}}` - rendered from `variables.REPOSITORY_PROFILE`.
- `{{USER_FOCUS}}` - rendered from `variables.USER_FOCUS`.

Example consumer-side rendering:

```bash
PR_NUMBER=123 python3 - <<'PY'
import json
import os
from pathlib import Path

profile = json.loads(Path("profiles/repositories/mantle-xyz/reth.json").read_text())
repository_profile = "\n".join(f"- {item}" for item in profile["variables"]["REPOSITORY_PROFILE"])
repo = profile["repository"]
pr_number = os.environ["PR_NUMBER"]

rendered = (Path("prompts/adversarial-review/codex-initial-review.md").read_text()
  .replace("{{PR_NUMBER}}", pr_number)
  .replace("{{REPO}}", repo)
  .replace("{{TARGET_LABEL}}", f"PR #{pr_number} in {repo}")
  .replace("{{REPOSITORY_PROFILE}}", repository_profile)
  .replace("{{USER_FOCUS}}", profile["variables"]["USER_FOCUS"]))
print(rendered)
PY
```

## Repository Profiles

Initial profiles:

- [Whisker17/mantle-agent-scaffold](profiles/repositories/Whisker17/mantle-agent-scaffold.json)
- [mantle-xyz/reth](profiles/repositories/mantle-xyz/reth.json)
- [mantlenetworkio/mantle-v2](profiles/repositories/mantlenetworkio/mantle-v2.json)
- [mantlenetworkio/op-geth](profiles/repositories/mantlenetworkio/op-geth.json)

This repository intentionally does not commit rendered artifacts. Consumers own
the final rendering step so they can decide how to inject runtime values such as
the pull request number.
