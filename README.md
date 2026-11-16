# github-actions

Reusable GitHub Actions workflows.

## `trivy.yml` (reusable workflow)

Trivy scan for Kubernetes/Helm misconfigurations and secrets. Reports in the run
summary and as a PR comment. Outputs: `total`, `critical`, `outcome`.

Inputs: `scan-ref` (default `.`), `severity` (default `HIGH,CRITICAL`).

## `telegram-notify` (action)

Generic Telegram message. Inputs: `bot-token`, `chat-id`, `message` (HTML),
optional `thread-id` and `link-text` (link to the run, default `Report`).
Skips quietly when token or chat id are empty.

## Example

```yaml
name: Trivy
on:
  pull_request:
  schedule:
    - cron: '0 3 * * 1'
  workflow_dispatch:

permissions:
  contents: read
  pull-requests: write

jobs:
  trivy:
    uses: petretiandrea/github-actions/.github/workflows/trivy.yml@v1

  notify:
    needs: trivy
    if: always() && (github.event_name == 'schedule' || github.event_name == 'workflow_dispatch')
    runs-on: ubuntu-24.04
    steps:
      - uses: petretiandrea/github-actions/telegram-notify@v1
        with:
          bot-token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          chat-id: ${{ secrets.TELEGRAM_CHAT_ID }}
          thread-id: ${{ secrets.TELEGRAM_THREAD_ID }}
          message: >-
            ${{ needs.trivy.outputs.outcome == 'success'
            && format('🔐 <b>Trivy</b> {0}: {1} findings ({2} critical)', github.repository, needs.trivy.outputs.total, needs.trivy.outputs.critical)
            || format('⚠️ <b>Trivy</b> {0}: scan did not complete', github.repository) }}
```

Secrets are set per repository or per organization.
