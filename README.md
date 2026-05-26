# codex-tast-run

Scheduled GitHub Actions workflow that keeps a Codex CLI `auth.json` access token from going stale.

Codex tokens expire after roughly 8 days of inactivity. This repo runs Codex headlessly on a cron, forces a token refresh, and writes the refreshed base64-encoded blob back to this repo's GitHub Secrets — so any workflow that pulls `CODEX_AUTH_JSON_BASE_64` always gets a fresh value.

## Schedule

- `cron: "0 3 * * 1,4"` — Mondays and Thursdays at 03:00 UTC.
- Manual: `gh workflow run "Refresh Codex auth.json" --repo cherian/codex-tast-run`

## Required secrets

| Secret | Purpose |
| --- | --- |
| `CODEX_AUTH_JSON_BASE_64` | Current Codex auth blob (base64). Rotated by each run. |
| `GH_PAT` | Fine-grained PAT scoped to this repo with **Secrets: Read and write** (and Metadata: Read). Used to overwrite `CODEX_AUTH_JSON_BASE_64`. |

`GITHUB_TOKEN` cannot write secrets, hence the PAT. GitHub Secrets are write-only — there is no `gh secret get`. The workflow's `jq empty` step is the only validity gate after rotation; if two consecutive runs go green, the round-trip is working.
