# CLI reference

> Status: target state. This is the planned v1 surface; none of it is implemented yet.

## Commands

| Command | Purpose |
|---------|---------|
| `gmail-brief-agent auth` | One-time OAuth consent flow; writes `token.json`. See the [setup guide](../how-to/set-up-google-oauth.md). |
| `gmail-brief-agent run morning` | Triage unprocessed mail, write the morning briefing (today's calendar), apply labels. |
| `gmail-brief-agent run afternoon` | Triage mail arrived since morning, write the afternoon briefing (tomorrow's calendar, unresolved items), apply labels. |
| `gmail-brief-agent eval export` | Pull a stratified sample of labeled historical email into golden-set JSONL for manual review. |
| `gmail-brief-agent eval run` | Run the eval suite against the configured dataset; print per-tier metrics and confusion matrix. |

Exit code is nonzero on any failure; a failed run writes no partial notes and applies no labels.

## Cron integration

The two crontab lines that replace the incumbent wrapper script:

```cron
0 7  * * * gmail-brief-agent run morning
0 16 * * * gmail-brief-agent run afternoon
```

(Invocation via uv and PATH details to be pinned during implementation.)

## Configuration

Config file outside the repo (planned default: `~/.config/gmail-brief-agent/config.toml`).

| Key | Meaning |
|-----|---------|
| `vault_path` | Obsidian vault root the notes are written into |
| `client_secret_path` | OAuth client credentials JSON |
| `token_path` | OAuth token store (created by `auth`) |
| `interest_profile_path` | Personal interest profile (lives in the private data repo checkout) |
| `classifier_model` | Stage 1 model id (Haiku-class default) |
| `composer_model` | Stage 2 model id (Sonnet-class default) |
| `per_run_cap` | Max emails triaged per run; remainder carries to next run |
| `otlp_endpoint` | OpenTelemetry collector endpoint (localhost) |
| `eval_data_path` | Golden-set location (private data repo checkout) |

## Labels

The Gmail labels the agent reads and writes are listed in [classification tiers](classification-tiers.md).
