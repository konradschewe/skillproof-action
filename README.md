# skillproof-action

GitHub Action for [skillproof](https://github.com/konradschewe/skillproof) — verify that Claude agent skills are correctly adopted in your codebase.

## Usage

### Minimal setup

```yaml
- uses: actions/checkout@v4

- uses: konradschewe/skillproof-action@v1
  with:
    skills-dir: .claude/plugins/my-skills/skills
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

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

The report is published to `https://<owner>.github.io/<repo>/skillproof/skillproof-report.html`.

> **Note:** GitHub Pages must be enabled (Settings → Pages → Source: Deploy from branch `gh-pages`).

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `skills-dir` | Path to directory containing `SKILL.md` files | required |
| `provider` | LLM provider: `anthropic` or `aicore` | `anthropic` |
| `anthropic-api-key` | Anthropic API key | — |
| `filter` | Only evaluate skills whose name contains this substring | — |
| `output-format` | `markdown`, `json`, `github-summary`, or `html` | `github-summary` |
| `output-file` | Write output to this file path | — |
| `skillproof-version` | Version of the skillproof CLI to use | `latest` |
| `publish-pages` | Publish HTML report to GitHub Pages | `false` |
| `pages-destination-dir` | Subdirectory on GitHub Pages | `skillproof` |

## Outputs

| Output | Description |
|--------|-------------|
| `report-path` | Path to the generated report file (set when `output-file` is given or `publish-pages` is `true`) |
