# skillproof-action

GitHub Action for [skillproof](https://github.com/konradschewe/skillproof) — verify that Claude agent skills are correctly adopted in your codebase.

See the [skillproof documentation](https://github.com/konradschewe/skillproof) for a full explanation of how evaluation works, adoption statuses, output formats, and caching.

---

## Quick start

### Anthropic

```yaml
- uses: actions/checkout@v4

- uses: konradschewe/skillproof-action@v1
  with:
    skills-dir: .claude/plugins/my-skills/skills
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

Results are automatically written to the Actions step summary.

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

---

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `skills-dir` | **yes** | — | Path to the directory containing `SKILL.md` files, searched recursively. |
| `provider` | no | `anthropic` | LLM provider: `anthropic` or `aicore`. |
| `anthropic-api-key` | no | — | Anthropic API key. Required when `provider` is `anthropic`. |
| `aicore-service-key` | no | — | SAP AI Core service key JSON from BTP (contains `clientid`, `clientsecret`, `url`, `tokenurl`). Required when `provider` is `aicore`. |
| `aicore-resource-group` | no | `default` | AI Core resource group (namespace where your model deployments live). |
| `filter` | no | — | Only evaluate skills whose name contains this substring. |
| `system-prompt` | no | — | Additional context appended to the evaluator's system prompt. Use to describe the nature of the repository (e.g. "this is a shared library, not a concrete agent"). |
| `strict` | no | `false` | Require exact APIs and patterns as specified in each skill. Without `strict`, functionally equivalent implementations are accepted. |
| `concurrency` | no | `1` | Number of skills to evaluate in parallel. |
| `output-format` | no | `github-summary` | Output format: `markdown`, `github-summary`, `json`, or `html`. |
| `output-file` | no | — | Write the report to this file path (relative to `GITHUB_WORKSPACE`). |
| `publish-pages` | no | `false` | Publish an HTML report to GitHub Pages. Requires `contents: write` permission and Pages enabled on the `gh-pages` branch. Forces `output-format: html`. |
| `pages-destination-dir` | no | `skillproof` | Subdirectory on GitHub Pages to publish to. The report is available at `https://<owner>.github.io/<repo>/<pages-destination-dir>/`. |
| `cache-dir` | no | `.skillproof-cache` | Directory for the evaluation cache. Persisted across runs via `actions/cache`. |

---

## Outputs

| Output | Description |
|---|---|
| `report-path` | Path to the generated report file. Set when `output-file` is given, or when `publish-pages` is `true`. |

---

## Provider details

### Anthropic

Pass `anthropic-api-key`. The action sets `ANTHROPIC_API_KEY` in the environment, which the `@anthropic-ai/sdk` picks up automatically.

Models used:
- Evaluator: `claude-sonnet`
- Explorer: `claude-haiku`

### SAP AI Core

Pass `aicore-service-key` (the full BTP service key JSON). The action sets `AICORE_SERVICE_KEY` in the environment; the `@sap-ai-sdk` authenticates via OAuth using the credentials in the JSON.

`aicore-resource-group` is optional — omit it to use `"default"`. The resource group identifies the AI Core namespace where your model deployments live; it is not part of the service key JSON.

Model deployments required in your AI Core instance:
- Evaluator: `anthropic--claude-4.6-sonnet`
- Explorer: `anthropic--claude-4.5-haiku`

---

## Publishing to GitHub Pages

To publish a standalone HTML report on a schedule:

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

> **Note:** GitHub Pages must be enabled: Settings → Pages → Source: Deploy from branch → `gh-pages`.

