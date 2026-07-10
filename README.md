# gmail-brief-agent

A purpose-built agent that triages a Gmail inbox and writes twice-daily briefing notes into an Obsidian vault, with an eval suite behind it.

**Status: design settled, implementation pending.** The v1 design is documented in [docs/](docs/README.md) (Diátaxis layout: architecture, design decisions, eval strategy, OAuth setup, CLI reference).

## Why

This replaces a prompt-only automation: a cron job that ran a headless Claude Code session against one large prompt, twice a day. That version works, but everything interesting about it is implicit. Rebuilding it as a first-class agent makes those parts explicit and measurable:

- Two fixed LLM stages with typed structured outputs instead of one free-running agent with shell access.
- Deterministic orchestration around them: fetching, extraction, rendering, and label writes are plain code.
- An eval harness built from months of real, labeled inbox traffic.
- Cost and latency budgets that are tracked, not guessed.

## What it will do

- Triage unprocessed Gmail into tiers (action required, interesting, routine, noise) against an interest profile.
- Summarize what matters and render morning and afternoon briefings as markdown notes.
- Track unresolved action items using Gmail labels as the single source of truth.

## Design principles

- **Small agentic surface.** The LLM is used where judgment is required: classification and summarization. Rendering notes and applying labels is deterministic code.
- **Evals before features.** Classification metrics per tier on a golden set, LLM-as-judge for summary quality, regression runs on every prompt or model change.
- **Performance is a feature.** Model tiering, prompt caching, and cost per briefing measured against the baseline it replaces.

## Privacy

The agent runs against a personal inbox. No real email content, no interest profile, and no personal data live in this repo. Eval fixtures are synthetic or anonymized.

## License

[MIT](LICENSE)
