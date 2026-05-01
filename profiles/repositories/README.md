# Repository Profiles

Repository profiles contain repo-specific context that gets rendered into generic
prompt templates.

Each supported repository should have exactly one JSON file:

```text
profiles/repositories/<owner>-<repo>.json
```

The JSON file should contain:

- `repository` - GitHub repository in `owner/name` form.
- `variables.USER_FOCUS` - one concise sentence describing review intensity and
  emphasis.
- `variables.REPOSITORY_PROFILE` - an array of durable repo facts, public
  contracts, important workflows, and risk areas.

Prompt templates reference these files through variables:

- `{{REPOSITORY_PROFILE}}`
- `{{USER_FOCUS}}`

CI/action rendering code should read the selected JSON profile, render
`variables.REPOSITORY_PROFILE` as a Markdown bullet list, replace the variables
in a versioned template, and then replace runtime variables such as
`{{PR_NUMBER}}` and `{{REPO}}`.
