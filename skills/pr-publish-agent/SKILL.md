---
name: pr-publish-agent
version: 1.0.0
description: Pushes the current branch, creates a PR linked to the configured tracker issue, comments the PR link on the issue, and moves the issue to In review.
---

# PR Publish Agent

## Purpose

Publish implementation work by pushing the branch, opening a PR linked to the configured tracker issue, and transitioning the issue to review.

## Runtime Configuration

- Read `/orchestra-config.json` from the repository root before starting.
- Read `issue_tracker` and use only the configured tracker MCP for ticket operations.
- Use the MCP mapped to `issue_tracker` in `orchestra-config.json`.
- If the configured issue tracker MCP is unavailable, stop immediately and do not proceed with the task.
- For every tracker comment/status update, include: `Skill-Version: pr-publish-agent@1.0.0`.
- Immediatly stop here if gh cli is not available.

## When to Invoke

- After implementation is complete
- When code is ready for review

## Required Inputs

- Parent issue ID
- Parent issue URL
- Current local branch with committed changes
- Repository default base branch (for example `main`)

## Outputs

- Branch pushed to remote
- Pull request created and linked to the configured tracker issue
- Parent issue comment containing PR URL
- Parent issue status moved to `In review`

## Procedure

1. Read `/orchestra-config.json` from the repository root, set the issue tracker context, and verify the configured tracker MCP is available.
2. Confirm there are committed changes on the current branch.
3. Push the current branch to origin.
4. Create a PR targeting the repository base branch.
5. Link the configured tracker issue in the PR title or PR body using the issue ID and URL.
6. Capture the PR URL.
7. Add a comment on the parent issue with the PR URL and a short review request note.
8. Move the parent issue status to `In review`.

## Suggested Command Pattern

- Push branch: `git push -u origin <branch>`
- Create PR: `gh pr create --base <base-branch> --head <branch> --title "<issue-id>: <short title>" --body "Tracker: <issue-url>"`

## Guardrails

- Do not create a PR if there are no committed changes.
- Do not move issue status to `In review` until PR creation succeeds.
- Ensure the PR references the correct tracker issue ID and URL.
- Do not run tracker operations unless the MCP for the configured `issue_tracker` is available.

## Handoff

Primary consumer: `pr-review-agent` and human reviewers.
