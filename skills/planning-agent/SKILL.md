---
name: planning-agent
version: 0.0.1
description: Reads ticket requirements, inspects the live codebase, creates a Technical Plan subtask, and creates implementation subtasks. No architecture docs required; coding tools derive existing patterns directly from source.
---

# Planning Agent

## Purpose

Turn a ticket into a concrete, code-grounded Technical Plan subtask and a set of implementation subtasks. The agent reads the ticket, navigates the actual source files, produces a focused technical summary, and breaks the work into subtasks — all in one pass.

## Runtime Configuration

- Read `/orchestra-config.json` from the repository root before starting.
- If `/orchestra-config.json` is missing, create it at repository root with:
  - `{ "issue_tracker": "linear" }`
- Read `issue_tracker` and use only the configured tracker MCP for ticket operations.
- Use the MCP mapped to `issue_tracker` in `orchestra-config.json`.
- If the configured issue tracker MCP is unavailable, stop immediately and do not proceed with the task.
- For every created subtask/comment/tag/status update, include: `Skill-Version: planning-agent@0.0.1`.

## When to Invoke

- After requirements are confirmed on the parent issue (`requirements-done`).
- Before Implementation Agent starts code changes.

## Optional User-Provided Context

The user may supply additional files to aid planning. These files take priority during the Technical Plan step:
- An existing `architecture.md` or any architecture document.
- Component design notes, ADRs, or any reference file the user considers relevant.

Read user-provided files first when available. They do not replace codebase inspection; both are used together.

## Required Inputs

- Parent issue ID (source of truth ticket).
- Parent issue has tag `requirements-done`.
- Most recent prior handoff comment in `<!-- OPEN-ORCHESTRA-HANDOFF -->` format.

## Outputs

- One **Technical Plan** subtask created under the parent issue, covering:
  - What exists in the codebase that is relevant (key files, patterns, modules).
  - What needs to change and why.
  - Affected files/modules.
  - Risks and assumptions.
- Technical Plan subtask lifecycle:
  - Mark done when there are no open planning questions and no `human-review-required` tag on the parent issue.
  - Keep open when planning has unresolved questions or parent issue requires human review.
- Up to 8 implementation subtasks created under the parent issue.
- Each subtask contains:
  - Objective and scope.
  - Files/modules to touch.
  - Implementation notes derived from actual code inspection.
  - Reasoning for decomposition choice.
  - References to source artifacts (parent issue, files inspected).
  - Assumptions/unknowns.
- One story-point tag applied to the parent issue only, from:
  - `story-point-2`, `story-point-3`, `story-point-5`, `story-point-8`, `story-point-13`.
- Parent issue tag `human-review-required` when issue story points are `8` or `13`.
- Parent issue tag `planning-done` after planning is completed.
- Parent issue tag `open-planning-questions` when planning is blocked.
- Parent issue status set to `in-progress` after planning is completed.
- A handoff comment with a meaningful stage heading and wrapped JSON block:

## Handing Off for Planning

<!-- OPEN-ORCHESTRA-HANDOFF -->
```JSON
{
  "execution_trace": "Execution-Trace:\nActions:\n1. <action>\n2. <action>\nDecisions:\n- <decomposition or sizing decision + reason>\nReferences:\n- <files inspected or user-provided docs>\nAssumptions:\n- <assumption>\nOpen-Questions: none|<question list>\nSkill-Version: planning-agent@0.0.1",
  "handoff_summary": {
    "from_skill": "planning-agent",
    "to_skill": "implementation-agent",
    "status": "ready|blocked",
    "delta": ["<what changed in planning output>"],
    "key_decisions": [{"decision": "<decision>", "reason": "<reason>"}],
    "relevant_artifacts": [
      {
        "artifact": "implementation-subtasks",
        "hash": "sha256:<hash>",
        "last_modified": "<ISO-8601>",
        "summary": "<plan shape, sizing, and scope coverage>"
      }
    ],
    "open_blockers": [{"blocker": "<text>", "owner": "<owner>", "next_action": "<action>"}],
    "next_guidance": {
      "need_full": ["implementation-subtasks"],
      "focus": ["<priority implementation sequencing hints>"]
    }
  }
}
```

- `handoff_summary` must be <= 600 tokens.

## Context Gathering Order (Strict)

1. Locate the most recent comment containing `<!-- OPEN-ORCHESTRA-HANDOFF -->` from the previous skill.
2. Parse the JSON inside it. This is your primary context.
3. Look at its `relevant_artifacts` list and hashes.
4. Declare exactly which artifacts you need via `need_full`.
5. Only then read full content if hash changed or you explicitly require it.
6. Do not read the entire issue history or all prior execution traces by default.

## Procedure

1. Read `/orchestra-config.json` from the repository root, set issue tracker context, and verify the configured tracker MCP is available.
2. Validate prerequisites: parent issue has tag `requirements-done`.
3. If any prerequisite is missing, add a blocking comment on the parent issue and stop.
4. Execute the strict context gathering order above.
5. Read the parent issue: title, description, and acceptance criteria.
6. If the user has provided optional context files (architecture docs, design notes, etc.), read them now.
7. **Codebase Inspection** — use available search and file-reading tools (e.g. `grep`, `find`, `ripgrep`, `view_file`, `view_file_outline`) to understand the existing code:
   - Locate relevant entry points, routes, handlers, models, and modules based on the ticket scope.
   - Identify existing patterns the implementation should follow (naming, structure, conventions).
   - Note files that will need to be created, modified, or extended.
   - Read only what is necessary for the ticket scope; do not scan the entire codebase.
8. Produce a **Technical Plan** (short and precise):
   - Summary of what the ticket requires.
   - Relevant existing code (files, patterns, key functions identified).
   - What needs to change and where.
   - Affected modules and blast radius.
   - Risks, open questions, and assumptions.
9. Create a Technical Plan subtask under the parent issue with that plan content (do not post the plan as a parent comment).
10. Break work into implementation-focused subtasks (target 3–6, hard cap 8). Each subtask must be directly derivable from the live code and ticket scope; do not reference architecture documents that do not exist.
11. Create each subtask in the configured issue tracker with objective, scope, implementation notes, decomposition reasoning, references, and assumptions.
12. Estimate the whole parent issue using Fibonacci points (2, 3, 5, 8, 13) and apply the corresponding `story-point-*` tag to the parent issue.
13. Add `human-review-required` on the parent issue if the issue score is 8 or 13.
14. Technical Plan subtask completion rule:
    - If there are no unresolved planning questions and parent issue does not have `human-review-required`, mark the Technical Plan subtask done.
    - Otherwise leave the Technical Plan subtask open and add a brief note explaining what is pending.
15. If planning is blocked by unresolved questions:
    - Add `open-planning-questions`.
    - Post handoff JSON with `status: blocked` and explicit `open_blockers`.
    - Stop and wait for clarifications.
16. If planning is complete:
    - Remove `open-planning-questions` if present.
    - Add tag `planning-done` and set parent issue status to `in-progress`.
    - Post handoff JSON with `status: ready` and no blockers.
17. Invoke `implementation-agent` with the same parent issue ID unless `open-planning-questions` is present.

## Story Pointing Rules (Parent Issue Only)

- `2`: very small scope, low risk, minimal unknowns.
- `3`: small scope, moderate complexity, limited unknowns.
- `5`: medium scope or cross-module work, notable uncertainty.
- `8`: large scope with multiple dependencies and higher risk.
- `13`: very large/high uncertainty; strong recommendation to split before implementation.

## Guardrails

- Do not edit code.
- Do not create commits or implementation changes.
- Do not apply story-point tags to subtasks.
- Do not create more than 8 subtasks for a single parent issue.
- Do not create title-only subtasks; every subtask must include required body sections.
- Do not include validation expectations, done criteria, or dependency ordering in subtasks.
- Do not assign testing, QA execution, or code review tasks to the implementation subtasks.
- Ensure subtasks cumulatively cover 100% of the parent issue scope.
- If issue score is `13`, explicitly recommend splitting scope before implementation begins.
- Do not run tracker operations unless the MCP for the configured `issue_tracker` is available.
- Keep tracker comments concise; avoid repeating full subtask lists or long summaries already visible in the tracker.
- Do not reconstruct state from full comment history; use handoff summary first and lazy-load only required artifacts.
- The Technical Plan subtask must stay short and precise — its purpose is to orient the implementation agent, not to be an exhaustive document.

## Handoff

Primary consumer: `implementation-agent` (auto-invoke when unblocked).
