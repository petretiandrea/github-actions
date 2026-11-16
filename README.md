# github-actions

Reusable GitHub Actions workflows.

## `trivy.yml`

Trivy scan for Kubernetes/Helm misconfigurations and secrets. Reports in the run
summary, as a PR comment, and on Telegram for scheduled/manual runs.

```yaml
name: Trivy
on:
  pull_request:
    paths: ['k8s/**']
  schedule:
    - cron: '0 3 * * 1'
  workflow_dispatch:

permissions:
  contents: read
  pull-requests: write

jobs:
  trivy:
    uses: petretiandrea/github-actions/.github/workflows/trivy.yml@v1
    with:
      scan-ref: k8s
    secrets: inherit
```

Inputs: `scan-ref` (default `.`), `severity` (default `HIGH,CRITICAL`),
`telegram-event-names` (default `schedule,workflow_dispatch`).

Secrets (optional, set per repo or per organization): `TELEGRAM_BOT_TOKEN`,
`TELEGRAM_CHAT_ID`, `TELEGRAM_THREAD_ID`. Without them the Telegram step is skipped.
