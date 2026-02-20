# openIT

`openIT` is a skill pack for Codex-style agents that run a Linear-driven software delivery workflow from planning to PR review and issue closure.

## What This Repo Does

This repository stores reusable agent skills in `skills/*/SKILL.md`.
Each skill defines:
- when it should run
- required inputs
- exact outputs
- guardrails and handoff to the next step

The goal is to make issue execution consistent, auditable, and fast.

## Available Skills (One-Line Each)

- `init-architect`: Builds a project `ARCHITECTURE.md` map and supporting architecture reference docs for downstream agents.
- `qa-agent`: Creates a Linear `qa-plan` subtask with ticket-native test cases from functional and technical requirements.
- `planning-agent`: Converts approved technical details into implementation-only subtasks and parent issue story points.
- `implementation-agent`: Implements `implement` subtasks in sequence, updates Linear status/tags, and records build/lint outcomes.
- `post-implementation-architect-agent`: Reviews implemented changes for architecture conformance, boundary safety, and maintainability risks.
- `pr-publish-agent`: Pushes the branch, opens a PR linked to the Linear issue, comments the PR URL, and moves issue status to review.
- `pr-review-agent`: Performs a risk-focused PR review with severity-ranked findings and a merge recommendation.
- `issue-summary-agent`: Produces the final closure summary with delivered scope, evidence, risks, and follow-up actions.

## Typical Workflow

1. `init-architect` (project onboarding or major architecture drift)
2. `qa-agent`
3. `planning-agent`
4. `implementation-agent`
5. `post-implementation-architect-agent`
6. `pr-publish-agent`
7. `pr-review-agent`
8. `issue-summary-agent`

## How To Use These Skills

1. Open this repo in Codex (or your agent runtime that supports `SKILL.md` workflows).
2. Ask the agent to use a specific skill by name (for example: `Use $planning-agent for LIN-123`).
3. Provide the inputs listed in that skill's `Required Inputs` section.
4. Execute the skill and follow its handoff to the next skill in the workflow.

## Repo Structure

```text
openIT/
  skills/
    <skill-name>/
      SKILL.md
  README.md
```

## Notes

- These skills assume Linear is the source of truth for issue state and comments.
- Some skills also expect Git and GitHub CLI (`gh`) access for branch/PR operations.
- If your team process differs, edit each `SKILL.md` guardrail/procedure to match your policy.
