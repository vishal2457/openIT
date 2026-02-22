# Open Orchestra

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

## Available Skills

- `init-architect`: Manually initializes or refreshes architecture artifacts under `architecture/` while preserving templates under `skills/init-architect/`.
- `requirements-ticket-agent`: Converts informal user requests into structured initial tickets by asking clarifying questions and outputting Title, Body, and Acceptance Criteria.
- `architect-agent`: Reads `architecture/architecture.md` and related `architecture/docs/*.md` files to create a `technical-details` subtask.
- `qa-agent`: Creates a Linear `qa-plan` subtask with ticket-native test cases from functional and technical requirements.
- `planning-agent`: Converts approved technical details into implementation-only subtasks and story points the parent issue.
- `implementation-agent`: Implements `implement` subtasks in sequence, updates Linear status/tags, and records build/lint outcomes.
- `pr-publish-agent`: Pushes the branch, opens a PR linked to the Linear issue, comments the PR URL, and moves issue status to review.
- `pr-review-agent`: Performs a risk-focused PR review with severity-ranked findings and a merge recommendation.
- `issue-summary-agent`: Produces the final closure summary with delivered scope, evidence, risks, and follow-up actions.

## Typical Workflow

1. Install skills in the target repository.
2. Use `requirements-ticket-agent` to turn initial requests into structured requirements.
3. Run `init-architect` manually (project onboarding or major architecture drift).
4. Manually kick off the first active stage (`requirements-ticket-agent` for net-new requests, or `architect-agent` for prepared tickets).
5. `requirements-ticket-agent` auto-invokes `architect-agent` unless requirements questions are open.
6. `architect-agent` auto-invokes `qa-agent` unless architecture questions are open.
7. `qa-agent` auto-invokes `planning-agent` unless QA questions are open.
8. `planning-agent` auto-invokes `implementation-agent` unless planning questions are open.
9. `implementation-agent` auto-invokes `pr-publish-agent` unless implementation questions are open.
10. `pr-publish-agent` auto-invokes `pr-review-agent` unless publish questions are open.
11. `pr-review-agent` loops back to `implementation-agent` when fixes are required, or completes when review is clean.

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
- `open-pr-publish-questions`
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
