# PR Adversarial Review Prompts

Three-pass adversarial review prompts for Mantle pull requests.

## Version

Current version: `v1`

## Execution Order

1. `v1/codex-initial-review.md`
   - Agent: Codex
   - Stage: initial adversarial review
   - Summary marker: `CODEX_INITIAL_REVIEW_SUMMARY`
2. `v1/claude-second-pass-review.md`
   - Agent: Claude
   - Stage: second-pass adversarial review and cross-review of Codex findings
   - Summary marker: `CLAUDE_REVIEW_SUMMARY`
3. `v1/codex-final-review.md`
   - Agent: Codex
   - Stage: final adversarial review and disposition of Claude findings
   - Summary marker: `CODEX_FINAL_REVIEW_SUMMARY`

## Required Placeholders

- `__PR_NUMBER__` - GitHub pull request number.
- `__REPO__` - GitHub repository in `owner/name` form.

## CI Contract

The rendered prompt should be passed to the matching agent after replacing all
required placeholders. The prompts expect GitHub CLI access to the target
repository and permission to create PR review comments.

The v1 Claude prompt was extracted as-is and uses
`mcp__github_inline_comment__create_inline_comment` for inline findings. A
workflow using that stage must provide the same tool or introduce a new prompt
version that posts Claude inline findings through another mechanism.

These prompts are serial by design. The second and third stages read review
comments created by earlier stages, so CI should not run all stages in parallel.
