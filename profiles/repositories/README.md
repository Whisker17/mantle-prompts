# Repository Profiles

Repository profiles contain repo-specific context that gets rendered into generic
prompt templates.

Each supported repository has exactly one JSON file:

```text
profiles/repositories/<account>/<repo>.json
```

The JSON file contains:

- `repository` - GitHub repository in `owner/name` form.
- `variables.USER_FOCUS` - one concise sentence describing review intensity and
  emphasis.
- `variables.REPOSITORY_PROFILE` - an array of durable repo facts, public
  contracts, important workflows, and risk areas.

Consumers read these files and render the selected prompt template in their own
workflow.
