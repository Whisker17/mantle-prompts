# PR Adversarial Review Prompts

Three-pass adversarial review prompts for pull requests.

## Templates

Templates live in `prompts/adversarial-review/`:

1. `codex-initial-review.md`
   - Agent: Codex
   - Stage: initial adversarial review
   - Summary marker: `CODEX_INITIAL_REVIEW_SUMMARY`
2. `claude-second-pass-review.md`
   - Agent: Claude
   - Stage: second-pass adversarial review and cross-review of Codex findings
   - Summary marker: `CLAUDE_REVIEW_SUMMARY`
3. `codex-final-review.md`
   - Agent: Codex
   - Stage: final adversarial review and disposition of Claude findings
   - Summary marker: `CODEX_FINAL_REVIEW_SUMMARY`

## Consumer Rendering

Downstream repositories render these templates with one repository profile and
the current PR number. This repository does not commit rendered prompts.

## Repository Profiles

Repository-specific context lives outside the prompt templates:

- `profiles/repositories/Whisker17/mantle-agent-scaffold.json`
- `profiles/repositories/mantle-xyz/reth.json`
- `profiles/repositories/mantlenetworkio/mantle-v2.json`
- `profiles/repositories/mantlenetworkio/op-geth.json`

Add another repo by creating one JSON profile with `variables.USER_FOCUS` and
`variables.REPOSITORY_PROFILE`, then registering it in `manifest.json`.

## CI Contract

The rendered prompt should be passed to the matching agent. The prompts expect
GitHub CLI access to the target repository and permission to create PR review
comments.

The Claude prompt uses `mcp__github_inline_comment__create_inline_comment` for
inline findings. A workflow using that stage must provide the same tool or
introduce a prompt variant that posts Claude inline findings through another
mechanism.

These prompts are serial by design. The second and third stages read review
comments created by earlier stages, so CI should not run all stages in parallel.
