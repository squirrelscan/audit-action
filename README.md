# squirrelscan audit — GitHub Action

Run a [squirrelscan](https://squirrelscan.com) website audit in CI and fail the
build when the results regress.

> **Source of truth.** This action is maintained in the squirrelscan monorepo
> and published to a public Marketplace repo (e.g. `squirrelscan/audit-action`).
> Reference it from your workflows by that public name + a version tag.

## Usage

```yaml
name: Audit
on: [pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: squirrelscan/audit-action@v1
        with:
          url: https://staging.example.com
          fail-on: "score<90,severity>=error"
          # optional — adds cloud enrichment (rendering, AI summary, tech detect)
          token: ${{ secrets.SQUIRREL_API_TOKEN }}
          # optional — post a summary comment back on the PR
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

The job fails (exit code `2`) when any `--fail-on` threshold trips. Deterministic
audits run with no token; a token only adds cloud features.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `url` | yes | — | URL to audit |
| `fail-on` | no | — | Threshold expression(s), comma-separated: `score<90`, `score:perf<80`, `severity>=error`, `errors>0`, `warnings>0` |
| `token` | no | — | `SQUIRREL_API_TOKEN` for cloud enrichment |
| `format` | no | `json` | Report output format |
| `output` | no | `squirrel-report.json` | Report file path |
| `max-pages` | no | — | Max pages to crawl |
| `version` | no | `latest` | squirrel CLI version to install (e.g. `v0.0.46`) for reproducible CI |
| `args` | no | — | Extra raw args appended to the audit command |
| `github-token` | no | — | Token used to post a PR summary comment |

## Outputs

| Output | Description |
|--------|-------------|
| `report` | Path to the generated report file |
| `exit-code` | `0` = passed, `2` = a `--fail-on` threshold tripped |

## Keep the report as an artifact

```yaml
      - uses: squirrelscan/audit-action@v1
        id: audit
        with:
          url: https://staging.example.com
          fail-on: "score<90"
      - if: always()
        uses: actions/upload-artifact@v4
        with:
          name: squirrel-report
          path: ${{ steps.audit.outputs.report }}
```

See the [CI integration guide](https://docs.squirrelscan.com/guides/ci) for more
recipes (GitLab, generic runners, token setup).
