---
name: implementation-agent
version: 1.0.0
description: Implements tracker subtasks tagged `implement` in sequence, updates task statuses, runs build/lint checks, and reports completion on the parent issue.
---

# Implementation Agent

## Purpose

Implement the parent issue by executing planned implementation subtasks one by one in the codebase.

## Runtime Configuration

- Read `/orchestra-config.json` from the repository root before starting.
- Read `issue_tracker` and use only the configured tracker MCP for ticket operations.
- Use the MCP mapped to `issue_tracker` in `orchestra-config.json`.
- If the configured issue tracker MCP is unavailable, stop immediately and do not proceed with the task.
- For every task/comment/status update written to the tracker, include: `Skill-Version: implementation-agent@1.0.0`.

## When to Invoke

- After planning is complete on the parent tracker issue
- Before PR publish and PR review

## Required Inputs

- Parent issue ID
- Parent issue status is `In-progress`
- Parent issue has tag `planning-done`
- Child subtasks tagged `implement`
- Technical context from the parent issue and implementation subtasks

## Outputs

- Code changes implementing all completable `implement` subtasks
- Git branch created as: `<issue-id>-<short-description>`
- Each completed subtask marked done in the configured issue tracker
- Comment on each incomplete subtask explaining why it was not completed
- Build and lint command results recorded in the configured issue tracker
- Parent issue comment with a short implementation summary
- Parent issue tag `implemented` added

## Procedure

1. Read `/orchestra-config.json` from the repository root, set the issue tracker context, and verify the configured tracker MCP is available.
2. Validate prerequisites: parent issue status is `In-progress` and parent has `planning-done` tag.
3. If prerequisites fail, add a blocking comment on the parent issue and stop.
4. Create a new git branch named `<issue-id>-<short-description>`.
5. Read only child subtasks tagged `implement` Implement them in the order they are listed.
6. Implement subtasks one by one, keeping each change aligned to the subtask scope.
7. After finishing a subtask commit it with proper commit message and mark it done in the configured tracker.
8. If a subtask cannot be completed, add a subtask comment with the blocker, impact, and next action.
9. Detect project build and lint commands from repository config (for example `package.json`, `Makefile`, or equivalent).
10. Run build and lint commands after implementation is complete.
11. Post a short parent-issue summary.
12. Add parent tag `implemented`.

## Guardrails

- Do not start implementation unless parent issue has `planning-done` and status `In-progress`.
- Do not execute subtasks that are not tagged `implement`.
- Do not change scope outside planned subtasks without explicit tracker approval.
- Do not leave incomplete subtasks without a blocker comment.
- Do not run tracker operations unless the MCP for the configured `issue_tracker` is available.

## Handoff

Primary consumer: `pr-publish-agent`.
