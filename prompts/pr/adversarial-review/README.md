# PR Adversarial Review Prompts

Three-pass adversarial review prompts for Mantle pull requests.

## Version

Current version: `v2`

`v1` is preserved as the original Mantle agent scaffold extraction.
`v2` is the generic template version with repository-specific context supplied
from one JSON profile per repository in `profiles/repositories/`.

## Execution Order

1. `v2/codex-initial-review.md`
   - Agent: Codex
   - Stage: initial adversarial review
   - Summary marker: `CODEX_INITIAL_REVIEW_SUMMARY`
2. `v2/claude-second-pass-review.md`
   - Agent: Claude
   - Stage: second-pass adversarial review and cross-review of Codex findings
   - Summary marker: `CLAUDE_REVIEW_SUMMARY`
3. `v2/codex-final-review.md`
   - Agent: Codex
   - Stage: final adversarial review and disposition of Claude findings
   - Summary marker: `CODEX_FINAL_REVIEW_SUMMARY`

## Required Placeholders

- `{{PR_NUMBER}}` - GitHub pull request number.
- `{{REPO}}` - GitHub repository in `owner/name` form.
- `{{TARGET_LABEL}}` - human-readable target label.
- `{{REPOSITORY_PROFILE}}` - durable repo-specific context.
- `{{USER_FOCUS}}` - concise repo-specific review emphasis.

## Repository Profiles

Repository-specific context lives outside the prompt templates. Initial profiles:

- `profiles/repositories/whisker17-mantle-agent-scaffold.json`
- `profiles/repositories/mantle-xyz-reth.json`
- `profiles/repositories/mantlenetworkio-mantle-v2.json`
- `profiles/repositories/mantlenetworkio-op-geth.json`

Add another repo by creating one JSON profile with `variables.USER_FOCUS` and
`variables.REPOSITORY_PROFILE`, then registering it in `manifest.json`.

## CI Contract

The rendered prompt should be passed to the matching agent after replacing all
required placeholders. The prompts expect GitHub CLI access to the target
repository and permission to create PR review comments.

The v1 Claude prompt was extracted as-is and uses
`mcp__github_inline_comment__create_inline_comment` for inline findings. A
workflow using that stage must provide the same tool or introduce a new prompt
version that posts Claude inline findings through another mechanism.

The v2 Claude prompt keeps the same inline-comment tool contract for now.

These prompts are serial by design. The second and third stages read review
comments created by earlier stages, so CI should not run all stages in parallel.
