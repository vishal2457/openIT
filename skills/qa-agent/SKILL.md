---
name: qa-agent
version: 1.0.0
description: Creates a QA planning subtask in the configured issue tracker tagged `qa-plan`, with test cases derived only from the ticket's functional and technical requirements.
---

# QA Agent

## Purpose

Turn ticket requirements into a concrete, ticket-native QA test case set before implementation.

## Runtime Configuration

- Read `/orchestra-config.json` from the repository root before starting.
- Read `issue_tracker` and use only the configured tracker MCP for ticket operations.
- Use the MCP mapped to `issue_tracker` in `orchestra-config.json`.
- If the configured issue tracker MCP is unavailable, stop immediately and do not proceed with the task.
- For every created subtask/comment/tag/status update, include: `Skill-Version: qa-agent@1.0.0`.

## When to Invoke

- After Requirement and Architect Agents finish
- Before Pre-Implementation Agent creates subtasks

## Required Inputs

- Parent issue ID (source of truth ticket)
- Functional requirements in the ticket
- Technical requirements and constraints from the tracker subtask tagged `technical-details`
- Acceptance criteria and business context in the ticket

## Outputs

- A new subtask under the parent issue, tagged `qa-plan`
- Test cases written directly in that QA subtask, including:
- Positive path cases
- Edge and failure cases
- Non-functional checks relevant to ticket scope
- Clear pass/fail expectations per test case
- Parent ticket comment linking the QA subtask and confirming QA planning completion

## Procedure

1. Read `/orchestra-config.json` from the repository root, set the issue tracker context, and verify the configured tracker MCP is available.
2. Read the parent issue only (description, acceptance criteria, comments, labels, linked requirement notes).
3. Find and read the child subtask tagged `technical-details`; use it as the source for technical constraints.
4. Translate ticket requirements into explicit testable behaviors.
5. Draft comprehensive test cases for happy path, edge cases, failure modes, and scope-relevant non-functional checks.
6. Create a subtask titled `QA Plan: <parent issue title>` and apply tag `qa-plan`.
7. Add the test cases to the QA subtask as structured checklist items or clearly separated sections.
8. Add a parent-issue comment linking the QA subtask and hand off to `pre-implementation-agent`.
9. Add a tag qa-plan-created to the parent ticket.

## Guardrails

- Do not rewrite product scope.
- Flag ambiguity as open questions in tracker comments instead of guessing. Add Tag open-qa-questions to the parent ticket.
- Separate must-pass checks from optional hardening checks.
- Do not create `QA_PLAN.md` or any external QA artifact.
- Do not read repository code or architecture documents for QA planning.
- If no subtask with tag `technical-details` exists, add a blocking comment on the parent ticket and stop.
- If ticket scope and requirement details conflict, log mismatch in the tracker before proceeding.
- Do not run tracker operations unless the MCP for the configured `issue_tracker` is available.

## Handoff

Add tag read-for-pre-implementation to the parent ticket. And change status to TODO.
