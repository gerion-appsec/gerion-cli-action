# gerion-cli-action

Official GitHub Action for running [Gerion](https://gerion.dev) security scans in your workflows.

## Usage

```yaml
- uses: gerion-appsec/gerion-cli-action@v1
  with:
    api-url: ${{ vars.GERION_API_URL }}
    api-key: ${{ secrets.GERION_API_KEY }}
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `scan-type` | No | `all` | `all` \| `secrets` \| `sca` \| `iac` \| `sast` |
| `code-path` | No | `.` | Path to scan, relative to `GITHUB_WORKSPACE` |
| `api-url` | No | — | Gerion API Gateway URL |
| `api-key` | No | — | Gerion M2M API key |
| `output-format` | No | — | `json` \| `markdown` \| `sarif` |
| `output-file` | No | — | Output file path (relative to `GITHUB_WORKSPACE`) |
| `log-level` | No | `info` | `debug` \| `info` \| `warning` \| `error` |
| `timeout` | No | `180` | Per-scanner timeout in seconds |

## Outputs

| Output | Description |
|---|---|
| `findings-file` | Absolute path to the output file (only set when `output-file` is provided) |
| `exit-code` | Exit code from the CLI (`0` = success, `1` = execution error) |

## Examples

### Minimal — send to Gerion API

```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: gerion-appsec/gerion-cli-action@v1
        with:
          api-url: ${{ vars.GERION_API_URL }}
          api-key: ${{ secrets.GERION_API_KEY }}
```

### SARIF upload to GitHub Advanced Security

```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - uses: gerion-appsec/gerion-cli-action@v1
        with:
          output-format: sarif
          output-file: gerion-results.sarif
      - uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: gerion-results.sarif
```

### JSON output + archive artifact

```yaml
- uses: gerion-appsec/gerion-cli-action@v1
  with:
    output-format: json
    output-file: gerion-results.json

- uses: actions/upload-artifact@v4
  with:
    name: gerion-results
    path: gerion-results.json
```

### Scan a subdirectory

```yaml
- uses: gerion-appsec/gerion-cli-action@v1
  with:
    code-path: backend/
    api-url: ${{ vars.GERION_API_URL }}
    api-key: ${{ secrets.GERION_API_KEY }}
```
