# Design decisions

> Record of the v1 design discussion (2026-07-10). Each entry lists the decision, the alternatives considered, and why the alternative lost. These are working decisions, not immutable ADRs; revisiting one means updating this file with the new reasoning.

## 1. Replace an existing automation, not greenfield

The project rebuilds a working automation: a cron job running a headless Claude Code session (`claude -p`) with one large prompt that triages Gmail and writes briefing notes into an Obsidian vault, twice daily. Rebuilding it as a purpose-built system makes the implicit parts explicit and measurable, and months of label history become eval data. A greenfield project would start with zero data and zero users.

## 2. v1 scope: triage-core cutover

**Decision.** v1 directly replaces the cron job: classify, apply labels, write briefing notes. It keeps the calendar sections, the unresolved-items section, and the exact current note formats. Backlog mode collapses to oldest-first with a per-run cap. Cutover is gated on evals (see [eval strategy](evals.md)).

**Alternatives.** Shadow mode first (run read-only alongside the incumbent, diff verdicts, cut over later) was the cautious option; rejected in favor of shipping the replacement directly and building the golden set retroactively from labeled history. Full parity cutover (including the elaborate backlog mode) was rejected as the longest path to anything live.

## 3. Runtime shape: cron-invoked CLI

**Decision.** A uv-managed Python CLI invoked by crontab, same shape as the incumbent minus the Claude Code wrapper. No resident process.

**Alternatives.** A Docker Compose service with an internal scheduler would match the VPS convention for services, but a twice-daily batch job is not a service: it would cost resident memory, mounts for the vault and tokens, and scheduling logic in the app. A cron-invoked container collects the costs of both shapes. Containerization stays open as a later step if ever needed.

## 4. Google access: direct API client, reused OAuth client

**Decision.** `google-api-python-client` with scopes `gmail.modify` and `calendar.readonly`. The existing Google Cloud project and the OAuth client already created for the gws CLI are reused; the agent runs its own consent once and keeps its own token.

**Alternatives.** Requiring the gws CLI as an authenticated prerequisite would have reused working auth but made every tool a subprocess call with JSON parsing, made tests mock subprocesses, and made the public repo depend on a CLI nobody else has configured. A hybrid (API for Gmail, gws for calendar) would maintain two auth mechanisms to save one trivial scope. A dedicated OAuth client ID for the agent was considered for cleaner separation and declined for pragmatism; the token, which is the actual credential, is per-app either way.

## 5. LLM shape: two-stage pipeline, no agent loop

**Decision.** Deterministic orchestration with two fixed LLM stages. Stage 1: one call per email, full content, structured verdict out. Stage 2: one call over the grouped verdicts for cross-email synthesis. No tool-calling loop anywhere.

**Alternatives.** A single agent with typed tools (the incumbent's shape, professionalized) demos more agency but collapses evals to end-to-end only, makes paths nondeterministic, and varies cost per run. A bounded mini-agent for classification (fetch-body tool, max 2-3 turns) was made redundant by the always-full-content decision below.

**Always full content.** The incumbent classified from metadata first to save cost inside one expensive session. Per-email calls on a cheap model delete that constraint: a full day of email classifies for around a cent, and the interest profile itself demands reading actual content. Deterministic HTML-to-text extraction keeps token counts sane.

## 6. Stage 2 synthesizes; templates render

**Decision.** The composer LLM does only what needs a cross-email view: dedupe stories covered by multiple newsletters, order by importance, write section prose. Deterministic templates render the notes and inject calendar, unresolved items, counts, and Gmail links from message IDs. The LLM never touches a link, path, or note structure.

**Alternatives.** Pure templating (no stage 2) keeps today's disconnected-blurbs quality. Letting the LLM write the whole note makes link and format integrity an LLM failure mode you must eval for instead of a thing the type system prevents.

## 7. Framework: Pydantic AI

**Decision.** Pydantic AI with Anthropic models underneath, structured outputs with validation retries, `instrument=True` for OpenTelemetry, pydantic-evals as the harness companion.

**Alternatives.** The raw Anthropic SDK offers the deepest low-level control at the cost of a few hundred lines of validation, retry, and instrumentation plumbing the framework provides. The Claude Agent SDK is a harness for the agentic loop this design deliberately does not have, and re-tells the wrapped-Claude story the project exists to move away from. With no tool loops, what is needed is structured outputs, model tiering, and instrumentation, which is exactly Pydantic AI's core.

## 8. Eval data: private companion repo

**Decision.** A private repo (`gmail-brief-agent-data`) holds the golden set as versioned JSONL plus the personal interest profile. It is cloned beside the public repo locally and in CI via deploy key. The public repo ships synthetic fixtures so anyone can run the harness.

**Alternatives.** Local-only data on the VPS has no versioning, no CI, and produces write-up numbers nobody can trace. Anonymizing real email for public committal is unreliable, and newsletter content is copyrighted; one slip is permanent.

## 9. Observability: self-hosted Grafana stack, in server-infra

**Decision.** Grafana + Tempo + Prometheus + OTel Collector as Compose services in the server-infra repo, on the shared web network behind Caddy, reusable by every project on the VPS. The agent emits OTLP to localhost and carries no observability infrastructure itself.

**Alternatives.** Logfire (hosted) has the best Pydantic AI integration but ships email content in traces to a third party and teaches no platform skills. Self-hosted Langfuse keeps privacy but wants Postgres, ClickHouse, Redis, and S3, a heavier stack than the system it observes, and overlaps pydantic-evals. The deciding fact: traces contain prompts, prompts contain email.

## 10. Project conventions: the Klassenzeit package

**Decision.** mise + uv, lefthook + cocogitto (conventional commits), CI with ruff, type checking, and pytest on every PR, branch protection on master, SHA-pinned third-party actions, secrets for the Anthropic key and the data-repo deploy key, plus an AI PR-review workflow.

**Alternatives.** pre-commit as hook runner is more widely recognized but duplicates tool-version management that mise already owns, and diverges from every other repo of the author. A minimal setup (uv + ruff + pytest only) was rejected because this repo's job is to be looked at.

## Parking lot

Deliberately out of v1 and of this design:

- Grafana exposure and auth in server-infra (Keycloak / oauth2-proxy).
- Briefing format evolution beyond the incumbent's layout.
- Stage 2 LLM-as-judge evals beyond a first faithfulness/dedupe version.
- Alerting on missed runs.
- Any Sieve integration (this project's classifier, composer, and eval harness are candidates to seed Sieve's digest pipeline later).
