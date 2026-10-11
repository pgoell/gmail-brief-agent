# Classification tiers

Every triaged email is assigned exactly one tier by the stage 1 classifier. The tier definitions below are the system's contract; the personal interest profile (private, see below) refines what counts as relevant within them but never changes their meaning.

## Tiers

| Tier | Definition | Briefing treatment |
|------|-----------|-------------------|
| **ActionRequired** | The user must DO something: reply, pay, confirm, decide, meet a deadline. Reading alone never qualifies. | Own section, top of the note, with reason and Gmail link. Tracked across runs until resolved. |
| **Interesting** | Substantive, applicable to the user's work or projects, and containing something genuinely new. All three, or it does not qualify. | Own section with a skeptical one-paragraph summary and source link. |
| **Routine** | Receipts, confirmations, automated notifications, calendar updates. Acknowledged but boring. | One-liner in the processed list. |
| **Noise** | Marketing, hype without substance, roundups rehashing known information, notification spam. | Counted, not listed. |

The classifier's posture is skeptical by design: assume marketing until proven otherwise, and judge newsletter editions by what is actually in them, not by the newsletter's reputation.

## Gmail labels

State lives in Gmail labels, nowhere else:

| Label | Meaning |
|-------|---------|
| `Claude/Processed` | Applied to every triaged email. Its absence defines "unprocessed". |
| `Claude/ActionRequired` | Applied to ActionRequired emails. The user removes it to mark the item resolved; the label is the single source of truth for the unresolved list. |
| `Claude/Interesting` | Applied to Interesting emails. |

Routine and Noise receive only `Claude/Processed`. The `Claude/` prefix is historical (the labels predate this project and are kept for continuity of the existing workflow and its months of history).

## The interest profile

What is "applicable" and "genuinely new" is personal. Those judgments live in an interest profile document that becomes the classifier's system prompt. It is private data and resides in the private data repo, referenced by config; the public repo never contains it. See [eval strategy](../explanation/evals.md) for how its effects are measured.
