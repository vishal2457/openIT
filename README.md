# openIT

`openIT` is a skill pack for Codex-style agents that run a ticket-driven software delivery workflow from planning to PR review and issue closure.

## What This Repo Does

This repository stores reusable agent skills in `skills/*/SKILL.md`.
Each skill defines:
- when it should run
- required inputs
- exact outputs
- guardrails and handoff to the next step

The goal is to make issue execution consistent, auditable, and fast.

## Available Skills (One-Line Each)

- `init-architect`: Manually initializes or refreshes architecture artifacts under `skills/architect-agent/`.
- `architect-agent`: Reads `skills/architect-agent/ARCHITECTURE.md` and related docs to create a `technical-details` subtask.
- `qa-agent`: Creates a Linear `qa-plan` subtask with ticket-native test cases from functional and technical requirements.
- `planning-agent`: Converts approved technical details into implementation-only subtasks and parent issue story points.
- `implementation-agent`: Implements `implement` subtasks in sequence, updates Linear status/tags, and records build/lint outcomes.
- `pr-publish-agent`: Pushes the branch, opens a PR linked to the Linear issue, comments the PR URL, and moves issue status to review.
- `pr-review-agent`: Performs a risk-focused PR review with severity-ranked findings and a merge recommendation.
- `issue-summary-agent`: Produces the final closure summary with delivered scope, evidence, risks, and follow-up actions.

## Typical Workflow

1. Install skills in the target repository.
2. Run `init-architect` manually (project onboarding or major architecture drift).
3. Run `architect-agent` per issue to create `technical-details`.
4. `qa-agent`
5. `planning-agent`
6. `implementation-agent`
7. `pr-publish-agent`
8. `pr-review-agent`
9. `issue-summary-agent`

## How To Use These Skills

1. Open this repo in Codex (or your agent runtime that supports `SKILL.md` workflows).
2. Ask the agent to use a specific skill by name (for example: `Use $planning-agent for LIN-123`).
3. Provide the inputs listed in that skill's `Required Inputs` section.
4. Execute the skill and follow its handoff to the next skill in the workflow.

## Repo Structure

```text
openIT/
  orchestra-config.json
  skills/
    <skill-name>/
      SKILL.md
    architect-agent/
      SKILL.md
      ARCHITECTURE.md
      docs/
  README.md
```

## Notes

- These skills read `orchestra-config.json` for runtime tracker selection (`linear` or `jira`).
- If the configured tracker MCP is unavailable, the skill must stop and not proceed with tracker operations.
- Each skill carries a `version` and stamps `Skill-Version: <skill-name>@<version>` in tracker artifacts for traceability.
- Some skills also expect Git and GitHub CLI (`gh`) access for branch/PR operations.
- If your team process differs, edit each `SKILL.md` guardrail/procedure to match your policy.
