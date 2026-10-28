# Scanverra Scan (GitHub Action)

Runs [`@scanverra/cli`](https://www.npmjs.com/package/@scanverra/cli) on your repository: hardcoded secrets, SAST issues, vulnerable dependencies, IaC misconfigurations and AI-agent config risks.

- **Your code stays on the runner.** The CLI downloads Scanverra's signed rules, scans locally and uploads only findings (file, line, rule, fix suggestion). File contents and secret values are never uploaded.
- **Fails the build** on findings at or above `fail-on` (default `high`), or, with `fail-on-new`, only when a pull request *adds* a critical or high issue compared with its target branch (the new-code gate).
- **GitHub code scanning:** uploads a SARIF report so findings show in the Security tab and on pull requests.

## Usage

1. Create an API key in Scanverra (Dashboard > API keys). The Free plan includes one; each uploaded scan counts toward the plan's monthly Repo Scanner scans.
2. Save it as a repository secret named `SCANVERRA_API_KEY`.
3. Add `.github/workflows/scanverra.yml`:

```yaml
on: [push, pull_request]
jobs:
  scanverra:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v4
      - uses: scanverra/scan-action@v1
        with:
          api-key: ${{ secrets.SCANVERRA_API_KEY }}
          fail-on: high
```

### Gate pull requests on new issues only

Existing issues on `main` shouldn't block every pull request. With `fail-on-new`, the job fails only when the change introduces a critical or high finding (or a vulnerable dependency) compared with the latest scan of the PR's target branch. Scan the target branch on push so there is a baseline:

```yaml
      - uses: scanverra/scan-action@v1
        with:
          api-key: ${{ secrets.SCANVERRA_API_KEY }}
          fail-on-new: ${{ github.event_name == 'pull_request' }}
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `api-key` | (required) | Scanverra API key |
| `path` | `.` | Folder to scan |
| `fail-on` | `high` | `critical`, `high`, `medium`, `low` or `none` |
| `category` | all | Comma-separated: `security`, `dependency`, `iac`, `quality` |
| `upload` | `true` | Send findings to your Scanverra dashboard |
| `sarif` | `true` | Upload SARIF to GitHub code scanning (needs `security-events: write`) |
| `fail-on-new` | `false` | Fail only on issues the change adds compared with the target branch's latest scan (overrides `fail-on`; needs `upload`) |
| `cli-version` | `0` | `@scanverra/cli` version or range |

## Outputs

| Output | Description |
| --- | --- |
| `sarif-file` | Path of the SARIF report |

## Publishing (maintainers)

This folder is the source of `github.com/scanverra/scan-action`. Copy `action.yml` and this README to the root of that repository, then tag a release and move the major tag:

```bash
git tag -a v1.0.0 -m "v1.0.0" && git tag -f v1 && git push origin v1.0.0 && git push -f origin v1
```
