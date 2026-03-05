# Open Orchestra

```bash
npx skills add https://github.com/vishal2457/open-orchestra/tree/main/skills
```

`Open Orchestra` is a skill pack for agents that run a ticket-driven software development workflow.

## What This Repo Does

This repository stores reusable agent skills in `skills/*/SKILL.md`. Our core philosophy is to implement AI agents in a way that mirrors how a real development team works — ticket by ticket, grounded in the actual codebase.

Each skill defines:
- when it should run
- required inputs
- exact outputs
- guardrails and handoff to the next step
- workflow tags and open-question behavior

The goal is to make issue execution consistent, auditable, and fast — without the overhead of maintaining external architecture documentation.

## Core Design Philosophy

**The code is the architecture document.**

Rather than requiring pre-built architecture snapshots that drift out of date, `planning-agent` reads the ticket requirements, navigates the live codebase using search and file-reading tools (`grep`, `ripgrep`, `find`, `view_file`), and produces a short, precise technical plan grounded in what the code actually looks like right now.

This means:
- The coding agent always sees the real, current state of the code.
- Less maintenance overhead; fewer stale context files confusing the agent.
- Implementation subtasks reference actual file paths and existing patterns — not guesses from an outdated doc.
- If required docs can be created for any project specific anti patterns and architecture decisions to tell the agent why it is implemented in a certain way.

**If you want architecture docs:** You can still create or maintain them. Pass them as optional context when invoking `planning-agent`, and the agent will read them alongside the live code. They inform but do not replace codebase inspection.

This workflow is designed to let agents work naturaly relying on their tools and reasoning capabilities. It lets agents rely on code rather then docs to derive existing functionality.

## Tracker Audit Trail Contract

All tracker-writing skills must leave an explicit audit trail. A run is incomplete unless all items below are present:

- Rich artifact content (issue/subtask body), not title-only updates.
- `Reasoning` section that explains why the agent chose the proposed scope/plan.
- `References` section listing exact inputs used (ticket fields, child tasks, files, PR links, commands when relevant).
- `Assumptions` section with unresolved unknowns and inferred decisions.
- Parent issue comment named `Execution-Trace` with:
  - actions performed (in order),
  - key decisions,
  - blockers/open questions,
  - `Skill-Version` stamp.

If any required section is missing, the skill must add a correction comment and keep the issue in blocked/open-questions state instead of marking the stage done.

## JSON Contract for Audit Trail and Handoff

In addition to readable markdown sections, each agent must emit machine-parseable JSON blocks for deterministic chaining and validation.

`Execution-Trace` JSON contract:

```json
{
  "agent": "planning-agent",
  "skill_version": "planning-agent@0.0.1",
  "timestamp_utc": "2026-02-22T10:30:00Z",
  "actions": ["read_parent_issue", "inspected_codebase", "created_technical_plan", "created_implement_subtasks", "estimated_story_points"],
  "decisions": ["split_api_and_ui_work", "deferred_non_blocking_cleanup"],
  "blockers": [],
  "open_questions": [],
  "references": ["LIN-123", "src/routes/v1/router.ts", "src/models/order.ts"]
}
```

`Workflow-Handoff` JSON contract:

```json
{
  "from_agent": "planning-agent",
  "to_agent": "implementation-agent",
  "status": "ready|blocked",
  "required_tags": ["planning-done"],
  "open_question_tags": [],
  "summary": "Technical plan posted. Implementation subtasks created and prioritized.",
  "artifacts": ["LIN-456", "LIN-457", "LIN-458"],
  "next_actions": ["implement_subtasks_in_order", "publish_pr_when_complete"]
}
```

Required rules:
- Use valid JSON objects (no comments, no trailing commas).
- Keep key names stable so downstream agents can parse consistently.
- If `status` is `blocked`, include at least one `open_question_tags` entry and describe the blocker in `summary`.

## Available Skills

- `requirements-ticket-agent`: Converts informal user requests into structured initial tickets by asking clarifying questions and outputting Title, Body, and Acceptance Criteria.
- `triage-agent`: Classifies an existing parent issue as `trivial|standard|complex`, suggests `skip_steps`, and routes to the correct next stage.
- `planning-agent`: Reads ticket requirements, inspects the live codebase, produces a short precise technical plan, and creates implementation subtasks. Optionally accepts user-provided architecture docs or reference files as additional context.
- `qa-agent`: Creates a `qa-plan` subtask with ticket-native test cases derived from functional requirements and the implementation subtasks. Runs after planning.
- `implementation-agent`: Implements `implement` subtasks in sequence, updates tracker status/tags, records build/lint outcomes, and publishes the PR when implementation is complete.
- `pr-review-agent`: Performs a risk-focused PR review with severity-ranked findings and a merge recommendation.

## Typical Workflow

```
requirements-ticket-agent  →  triage-agent  →  planning-agent  →  implementation-agent  →  pr-review-agent
                                                      ↓ (parallel)
                                                  qa-agent
```

1. Install skills in the target repository.
2. Create or identify a parent issue.
3. For net-new requests without a usable parent issue body, run `requirements-ticket-agent` first to create/normalize requirements on the parent issue.
4. Run `triage-agent` once a parent issue exists to classify execution track and propose routing.
5. `triage-agent` routes to `requirements-ticket-agent` (if requirements are missing) or directly to `planning-agent` (if requirements are done).
6. `planning-agent` reads the ticket, inspects the live codebase, creates a Technical Plan subtask, and creates implementation subtasks.
7. `qa-agent` runs after planning completes and creates a QA plan subtask. QA is a parallel quality gate — it does not block implementation.
8. `planning-agent` auto-invokes `implementation-agent` when planning is complete.
9. `implementation-agent` publishes the PR as part of the same run and auto-invokes `pr-review-agent` unless implementation questions are open.
10. `pr-review-agent` loops back to `implementation-agent` when fixes are required, or completes when review is clean.

### Optional: Architecture Docs as Extra Context

If your project maintains architecture documentation, you can pass it to `planning-agent` as optional context:

```
Use $planning-agent for LIN-123 with context: architecture/architecture.md
```

The planning agent will read the provided file alongside the live codebase. This is useful for large projects where you want to capture high-level system boundaries, security constraints, or cross-service contracts.

## Workflow Tag Contract

Parent issue completion tags:
- `requirements-done`
- `planning-done`
- `qa-done`
- `implementation-done`
- `pr-published`
- `pr-reviewed`

Parent issue open-question tags (agent-scoped):
- `open-requirements-questions`
- `open-planning-questions`
- `open-qa-questions`
- `open-implementation-questions`
- `open-pr-review-questions`

Every agent writes a `Workflow-Handoff` comment block with a stage-specific heading and the next agent reads only the previous agent's handoff block for open-question checks.

> **Migration note:** Tags `architecture-done`, `qa-plan-created`, and `implemented` are still accepted as legacy prerequisites during transition to the new workflow.

## Project Setup

The workflow is designed to be zero-config. Agents resolve the target ticket from current conversation context or the latest workflow handoff comment.

If no ticket reference is available for a ticket-scoped run, the agent asks for the ticket reference and waits before continuing.

## How To Use These Skills

1. Open this repo in Codex (or your agent runtime that supports `SKILL.md` workflows).
2. Kick off the workflow manually with the first needed skill (for example: `Use $planning-agent for LIN-123`).
3. Provide the inputs listed in that skill's `Required Inputs` section.
4. Execute the skill; downstream agents chain automatically unless an `open-*-questions` tag blocks progression.

## Repo Structure

```text
openIT/
  skills/
    requirements-ticket-agent/
      SKILL.md
    triage-agent/
      SKILL.md
    planning-agent/
      SKILL.md
    qa-agent/
      SKILL.md
    implementation-agent/
      SKILL.md
    pr-review-agent/
      SKILL.md
  README.md
```

## Notes

- If a ticket-scoped run has no ticket reference in context, the skill asks for it and stops until provided.
- If the required tracker MCP is unavailable, the skill must stop and not proceed with tracker operations.
- Each skill carries a `version` and stamps `Skill-Version: <skill-name>@<version>` in tracker artifacts for traceability.
- Some skills also expect Git and GitHub CLI (`gh`) access for branch/PR operations.
- If your team process differs, edit each `SKILL.md` guardrail/procedure to match your policy.