# GitHub Watcher Action

GitHub Watcher Action is a GitHub action for setting up custom reactions on various triggers
using [GitHub Watcher](https://github.com/ovsds/github-watcher).

[Cache](https://github.com/actions/cache) is used for storing the state.

## Usage

### Add a workflow

```yaml
name: Github Watcher

concurrency:
  group: ${{ github.workflow }} # allows only one run at a time

on:
  schedule:
    - cron: "*/5 * * * *" # every 5 minutes
  workflow_dispatch: # allows manual trigger

jobs:
  run-github-watcher:
    runs-on: ubuntu-20.04

    permissions:
      contents: read

    steps:
      - name: Checkout config file
        uses: actions/checkout@v4
        with:
          sparse-checkout: |
            .github/github-watcher-config.yaml
          sparse-checkout-cone-mode: false

      - name: Run
        uses: ovsds/github-watcher-action@v1
        with:
          config_path: .github/github-watcher-config.yaml
          env_variables: |
            GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }}
            TELEGRAM_TOKEN=${{ secrets.TELEGRAM_TOKEN }}
            TELEGRAM_CHAT_ID=${{ secrets.TELEGRAM_CHAT_ID }}
```

### Add a config file

```yaml
tasks:
  - id: example_task
    triggers:
      - id: example_trigger
        type: github
        token_secret:
          type: env
          key: GITHUB_TOKEN
        owner: ovsds
        subtriggers:
          - type: repository_issue_created
          - type: repository_pr_created
          - type: repository_failed_workflow_run
    actions:
      - id: example_action
        type: telegram_webhook
        chat_id_secret:
          type: env
          key: TELEGRAM_CHAT_ID
        token_secret:
          type: env
          key: TELEGRAM_TOKEN
```

More information on config can be found in
[GitHub Watcher Config](https://github.com/ovsds/github-watcher/blob/main/backend/README.md#config)

## Inputs

| Name                 | Description                                                                             | Default                            |
| -------------------- | --------------------------------------------------------------------------------------- | ---------------------------------- |
| `version`            | GitHub Watcher version to use.                                                          | `0.3.1`                            |
| `config_path`        | Path to the config file.                                                                |                                    |
| `env_variables`      | Environment variables to set.                                                           |                                    |
| `state_path`         | Path to the state folder. Must be absolute and independent from the runner's workspace. | `/tmp/github-watcher-action/state` |
| `env_path`           | Path to the temporary env file used to store env_variables during run.                  | `.env`                             |
| `state_cache_prefix` | Prefix for the state cache key.                                                         | `github-watcher-state`             |

## GitHub Watcher Settings

Settings file can be found in [settings.yaml](settings.yaml). Basically, it contains:

- `config_backend` - `yaml_file`
- `state_backend` - `local_dir` which is stored in GitHub cache.
- `queue_backend` - `in_memory`

## Development

### Global dependencies

- node

### Taskfile commands

For all commands see [Taskfile](Taskfile.yaml) or `task --list-all`.

## License

[MIT](LICENSE)
