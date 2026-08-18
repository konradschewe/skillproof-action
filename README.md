# skillproof-action

GitHub Action for [skillproof](https://github.com/konradschewe/skillproof) — verify that Claude agent skills are correctly adopted in your codebase.

## Usage

### Anthropic

```yaml
- uses: actions/checkout@v4

- uses: konradschewe/skillproof-action@v1
  with:
    skills-dir: .claude/plugins/my-skills/skills
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### SAP AI Core

```yaml
- uses: actions/checkout@v4

- uses: konradschewe/skillproof-action@v1
  with:
    skills-dir: .claude/plugins/my-skills/skills
    provider: aicore
    aicore-service-key: ${{ secrets.AICORE_SERVICE_KEY }}
    # aicore-resource-group: default   # optional, defaults to "default"
```

`AICORE_SERVICE_KEY` must be the full service key JSON from BTP, containing `clientid`, `clientsecret`, `url`, and `tokenurl`. Store it as a repository or organization secret.

Results are automatically written to the GitHub Actions step summary.

### With GitHub Pages report

Publish a standalone HTML report to GitHub Pages on a schedule:

```yaml
# .github/workflows/skillproof.yml
name: Skillproof

on:
  schedule:
    - cron: "0 6 * * 1"  # every Monday at 6am
  workflow_dispatch:

permissions:
  contents: write

jobs:
  skillproof:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: konradschewe/skillproof-action@v1
        with:
          skills-dir: .claude/plugins/my-skills/skills
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          publish-pages: true
```

The report is published to `https://<owner>.github.io/<repo>/skillproof/`.

> **Note:** GitHub Pages must be enabled (Settings → Pages → Source: Deploy from branch `gh-pages`).

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `skills-dir` | Path to directory containing `SKILL.md` files | required |
| `provider` | LLM provider: `anthropic` or `aicore` | `anthropic` |
| `anthropic-api-key` | Anthropic API key (required when `provider` is `anthropic`) | — |
| `aicore-service-key` | SAP AI Core service key JSON from BTP (required when `provider` is `aicore`) | — |
| `aicore-resource-group` | SAP AI Core resource group | `default` |
| `filter` | Only evaluate skills whose name contains this substring | — |
| `output-format` | `markdown`, `json`, `github-summary`, or `html` | `github-summary` |
| `output-file` | Write output to this file path | — |
| `publish-pages` | Publish HTML report to GitHub Pages | `false` |
| `pages-destination-dir` | Subdirectory on GitHub Pages | `skillproof` |

## Outputs

| Output | Description |
|--------|-------------|
| `report-path` | Path to the generated report file (set when `output-file` is given or `publish-pages` is `true`) |

## Provider details

### Anthropic

Set `provider: anthropic` (default) and pass `anthropic-api-key`. The action sets `ANTHROPIC_API_KEY` in the environment, which the `@anthropic-ai/sdk` picks up automatically.

### SAP AI Core

Set `provider: aicore` and pass `aicore-service-key`. The action sets `AICORE_SERVICE_KEY` in the environment; the `@sap-ai-sdk` reads it and authenticates via OAuth using the credentials in the JSON.

`aicore-resource-group` is optional. If omitted, skillproof uses `"default"`. The resource group is not part of the service key JSON — it identifies the AI Core namespace where your model deployments live.

The AI Core provider uses these model deployments:
- Evaluator: `anthropic--claude-4.6-sonnet`
- Explorer: `anthropic--claude-4.5-haiku`

These must be deployed in your AI Core instance under the configured resource group.
