# Open Orchestra

```bash
npx skills add https://github.com/vishal2457/open-orchestra/tree/main/skills
```

`Open Orchestra` is a skill pack for agents that run a ticket-driven software delivery workflow from planning to PR review and issue closure.

## What This Repo Does

This repository stores reusable agent skills in `skills/*/SKILL.md`. Our core philosophy is to implement AI agents in a way that models how a development team works around software tools in an agile environment. By defining and following every step of the software organization's lifecycle, we can successfully scale agentic development workflows in larger projects and across bigger teams.

Each skill defines:
- when it should run
- required inputs
- exact outputs
- guardrails and handoff to the next step
- workflow tags and open-question behavior

The goal is to make issue execution consistent, auditable, and fast.

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
  "skill_version": "planning-agent@1.2.0",
  "timestamp_utc": "2026-02-22T10:30:00Z",
  "actions": ["read_parent_issue", "created_implement_subtasks", "estimated_story_points"],
  "decisions": ["split_api_and_ui_work", "deferred_non_blocking_cleanup"],
  "blockers": [],
  "open_questions": [],
  "references": ["LIN-123", "LIN-456", "src/routes/v1/router.ts"]
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
  "summary": "Implementation subtasks are created and prioritized.",
  "artifacts": ["LIN-456", "LIN-457", "LIN-458"],
  "next_actions": ["implement_subtasks_in_order", "publish_pr_when_complete"]
}
```

Required rules:
- Use valid JSON objects (no comments, no trailing commas).
- Keep key names stable so downstream agents can parse consistently.
- If `status` is `blocked`, include at least one `open_question_tags` entry and describe the blocker in `summary`.

## Available Skills

- `init-architect`: Manually initializes or refreshes architecture artifacts under `architecture/` while preserving templates under `skills/init-architect/`.
- `triage-agent`: Classifies an existing parent issue as `trivial|standard|complex`, suggests `skip_steps`, and routes to the correct next stage.
- `requirements-ticket-agent`: Converts informal user requests into structured initial tickets by asking clarifying questions and outputting Title, Body, and Acceptance Criteria.
- `architect-agent`: Reads `architecture/architecture.md` and related `architecture/docs/*.md` files to create a `technical-details` subtask.
- `qa-agent`: Creates a Linear `qa-plan` subtask with ticket-native test cases from functional and technical requirements.
- `planning-agent`: Converts approved technical details into implementation-only subtasks and story points the parent issue.
- `implementation-agent`: Implements `implement` subtasks in sequence, updates Linear status/tags, records build/lint outcomes, and publishes the PR when implementation is complete.
- `pr-review-agent`: Performs a risk-focused PR review with severity-ranked findings and a merge recommendation.

## Typical Workflow

1. Install skills in the target repository.
2. Create or identify a parent issue.
3. For net-new requests without a usable parent issue body, run `requirements-ticket-agent` first to create/normalize requirements on the parent issue.
4. Run `init-architect` manually (project onboarding or major architecture drift).
5. Run `triage-agent` once a parent issue exists to classify execution track and propose routing.
6. `triage-agent` routes to `requirements-ticket-agent`, `architect-agent`, or `planning-agent` based on issue readiness/tags.
7. `requirements-ticket-agent` auto-invokes `architect-agent` unless requirements questions are open.
8. `architect-agent` auto-invokes `qa-agent` unless architecture questions are open.
9. `qa-agent` auto-invokes `planning-agent` unless QA questions are open.
10. `planning-agent` auto-invokes `implementation-agent` unless planning questions are open.
11. `implementation-agent` publishes the PR as part of the same run and auto-invokes `pr-review-agent` unless implementation questions are open.
12. `pr-review-agent` loops back to `implementation-agent` when fixes are required, or completes when review is clean.

## Workflow Tag Contract

Parent issue completion tags:
- `requirements-done`
- `architecture-done`
- `qa-done`
- `planning-done`
- `implementation-done`
- `pr-published`
- `pr-reviewed`

Parent issue open-question tags (agent-scoped):
- `open-requirements-questions`
- `open-architecture-questions`
- `open-qa-questions`
- `open-planning-questions`
- `open-implementation-questions`
- `open-pr-review-questions`

Every agent writes a `Workflow-Handoff` comment block and the next agent reads only the previous agent's handoff block for open-question checks.
Migration note: `qa-plan-created` and `implemented` are still accepted as legacy prerequisites while transitioning to `qa-done` and `implementation-done`.

## How To Use These Skills

1. Open this repo in Codex (or your agent runtime that supports `SKILL.md` workflows).
2. Kick off the workflow manually with the first needed skill (for example: `Use $architect-agent for LIN-123`).
3. Provide the inputs listed in that skill's `Required Inputs` section.
4. Execute the skill; downstream agents chain automatically unless an `open-*-questions` tag blocks progression.

## Repo Structure

```text
openIT/
  orchestra-config.json
  architecture/
    architecture.md
    docs/
  skills/
    <skill-name>/
      SKILL.md
    init-architect/
      SKILL.md
      ARCHITECTURE.md (template)
    architect-agent/
      SKILL.md
  README.md
```

## Notes

- These skills read `orchestra-config.json` for runtime tracker selection (`linear` or `jira`).
- If the configured tracker MCP is unavailable, the skill must stop and not proceed with tracker operations.
- Each skill carries a `version` and stamps `Skill-Version: <skill-name>@<version>` in tracker artifacts for traceability.
- Some skills also expect Git and GitHub CLI (`gh`) access for branch/PR operations.
- If your team process differs, edit each `SKILL.md` guardrail/procedure to match your policy.

## Documentation

- Workflow token estimate checkpoints: [`docs/workflow-token-estimates.md`](docs/workflow-token-estimates.md)
