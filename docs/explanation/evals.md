# Eval strategy

> Status: target state. The harness described here gates the cutover from the incumbent automation; it is built before the pipeline replaces anything.

## Why evals lead this project

The incumbent automation has been labeling real inbox traffic since March 2026. Every email carries its verdict (`Claude/Processed` plus optionally `Claude/ActionRequired` or `Claude/Interesting`), and the user's corrections are embedded in that history: labels removed mean false positives, labels added mean misses. That makes classification quality measurable before the replacement writes a single label, which is the reverse of the usual order (ship first, measure never).

## The golden set

- **Source.** An `eval export` subcommand pulls a stratified sample of historical emails with their final label state: extracted text, metadata, and the tier implied by the labels.
- **Upgrade to ground truth.** The exported labels are what the old prompt decided, not what is true. A deliberate manual review pass corrects them; the reviewed verdict is the gold label. This review is done by the user, case by case, and is treated as part of the engineering work, not a chore.
- **Storage.** Versioned JSONL in the private `gmail-brief-agent-data` repo, alongside the personal interest profile. Never in the public repo; see [design decision 8](design-decisions.md#8-eval-data-private-companion-repo).
- **Public fixtures.** The public repo ships a small synthetic fixture set with the same schema, so the harness runs end to end for anyone who clones it.

## Harness

pydantic-evals (the Pydantic AI companion): datasets of cases, evaluators over the pipeline's structured outputs, runs linked to OpenTelemetry traces. The harness lives in the public repo; only the data is private.

## Metrics

Stage 1 (classification) is scored per tier with precision, recall, and a confusion matrix. The tiers are not equally important:

- **ActionRequired recall is the headline metric.** Missing something the user must act on is the worst failure the system can produce.
- **Noise precision matters second.** Every false "interesting" is an attention tax on the person the system exists to protect.

Stage 2 (composition) is judged later and more loosely: an LLM-as-judge first version scoring faithfulness to the stage 1 verdicts and correctness of cross-newsletter dedupe. It does not gate v1 cutover.

## Baseline and cutover gate

The incumbent's own labels on the golden set form the baseline. The gate for replacing the cron job: the new stage 1 matches or beats the baseline, with ActionRequired recall weighted first. Until then the old automation keeps running.

## When evals run

- Locally, on demand, during development.
- In CI on pull requests that touch prompts or model configuration (path-filtered, because eval runs cost real API money), plus a manual dispatch trigger.
- Regression runs record their numbers so prompt and model changes carry evidence, not vibes.
