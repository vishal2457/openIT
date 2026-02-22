# Workflow Token Consumption Estimates

This document provides rough token usage estimates for the Open Orchestra workflow described in the repository. These are planning-level estimates, not exact metered values.

## Assumptions

- Estimates are for one parent issue moving through the standard flow in `README.md`.
- `triage-agent` always runs once per parent issue.
- `requirements-ticket-agent` may be skipped when the parent issue is already well-structured.
- PR publishing is included in `implementation-agent` (no separate `pr-publish-agent` run).
- "Tokens" means total model tokens (input + output) consumed across each agent run.
- Token usage includes reading tracker artifacts and required handoff blocks.
- Tool execution costs (Git, CLI, network latency, etc.) are excluded unless text is fed back into the model.
- Numbers are intentionally conservative and rounded.

## Checkpoints and Estimated Tokens

| Checkpoint | Typical Activity | Estimated Tokens (Low / Mid / High) |
| --- | --- | --- |
| 1. Triage routing (`triage-agent`) | Classify complexity, set routing tags, write execution trace + handoff | 700 / 1,200 / 2,200 |
| 2. Requirements drafting (`requirements-ticket-agent`, conditional) | Clarification questions, ticket synthesis, acceptance criteria, execution trace, handoff | 2,500 / 4,500 / 8,000 |
| 3. Architecture handoff (`architect-agent`) | Read architecture docs + parent issue, create `technical-details`, trace + handoff | 2,000 / 3,500 / 6,500 |
| 4. QA planning (`qa-agent`) | Read requirements + `technical-details`, generate QA plan, trace + handoff | 1,800 / 3,000 / 5,000 |
| 5. Implementation planning (`planning-agent`) | Read parent + QA + technical details, create up to 8 subtasks, estimate points, trace + handoff | 2,500 / 4,000 / 7,000 |
| 6. Implementation + PR publish (`implementation-agent`) | Read subtasks/context, implement task-by-task, summarize build/lint outcomes, publish PR, handoff | 5,000 / 9,800 / 19,000 |
| 7. PR review (`pr-review-agent`) | Read issue + technical details + PR diff, risk review, tracker/PR comments, handoff | 3,000 / 6,000 / 12,000 |

## End-to-End Totals

### Clean path with requirements step (no blocked questions, no review rework loop)

- **Low:** ~17,500 tokens
- **Mid:** ~32,000 tokens
- **High:** ~59,700 tokens

### Clean path without requirements step (triage routes directly to architecture/planning)

- **Low:** ~15,000 tokens
- **Mid:** ~27,500 tokens
- **High:** ~51,700 tokens

### With one rework loop (PR review finds fixes; implementation + review run again)

Additional tokens for one loop:
- **Low:** +8,000
- **Mid:** +15,800
- **High:** +31,000

Rework-path totals:
- **Low:** ~25,500 tokens
- **Mid:** ~47,800 tokens
- **High:** ~90,700 tokens

## Where Token Spikes Usually Happen

1. `implementation-agent`:
Large input context (multiple subtasks + code diff summaries), iterative reasoning, and PR publication payload generation.
2. `pr-review-agent`:
PR diff size and cross-checking against acceptance criteria + `technical-details`.
3. `requirements-ticket-agent`:
Clarification rounds can increase output significantly when scope is ambiguous.

## Blocked/Open-Question Overhead

Each blocked state (`open-*-questions`) typically adds:
- **~500 to 1,800 tokens** per additional clarification cycle

Reason:
- Re-reading latest `Workflow-Handoff`
- Writing blocker comments
- Producing a revised handoff after clarification

## Quick Capacity Rule-of-Thumb

For planning budget per issue:

- **Straight-through issue:** budget **35k tokens**
- **Issue likely to need one review loop:** budget **50k tokens**
- **Complex/high-ambiguity issue:** budget **90k tokens**

These budgets align with the midpoint estimates and provide safety margin for variance in prompt size and artifact verbosity.
