---
name: init-architect
version: 1.0.0
description: This skill generates a structured `ARCHITECTURE.md` for an existing codebase. It is the **first skill invoked** in the development workflow and its output becomes the reference document that all subsequent agents (Architect, Pre-Implementation, PR Review) consume. The goal is a living document — concise, opinionated, and scoped to what agents actually need. Not a wiki. Not a tutorial. A precise map of the system.
---

# Architecture Mapping Agent

## Purpose

This skill generates a structured `ARCHITECTURE.md` for an existing codebase. It is the **first skill invoked** in the development workflow and its output becomes the reference document that all subsequent agents (Architect, Pre-Implementation, PR Review) consume.

The goal is a living document — concise, opinionated, and scoped to what agents actually need. Not a wiki. Not a tutorial. A precise map of the system.

## Runtime Configuration

- Load `config.md` before starting.
- Read `issue_tracker` to drive any generated skill behavior for ticket operations.
- Generated ticket-facing skills must use the MCP mapped to `issue_tracker` in `config.md`.
- Generated ticket-facing skills must stop immediately when the configured tracker MCP is unavailable.
- Generated ticket comments/subtasks must include the skill version metadata for traceability.

---

## When to Invoke

- When onboarding a new project into the workflow for the first time
- When the codebase has changed significantly and the existing `ARCHITECTURE.md` is stale
- When the Architect Agent flags that the architecture doc is insufficient for decision-making

**Invocation**: Run this skill manually by pointing it at the root of the repository.

---

## Inputs

- Path to the repository root
- (Optional) Existing `ARCHITECTURE.md` to update rather than generate fresh
- (Optional) Stack or framework hints from the human if auto-detection is ambiguous

---

## Outputs

- `ARCHITECTURE.md` written to the repository root — the index, fast to read
- `docs/arch/*.md` — one reference file per complex domain, linked from the index
- `skills/architect/SKILL.md` — a project-specific Architect Agent skill, pre-configured with this project's domains, sensitive areas, and reference file paths baked in — no generic placeholders
- A brief terminal summary of what was discovered, which reference files were created, and any ambiguities flagged
---

## Procedure

### Step 1 — Discover the project structure

Explore the repository without reading every file. The goal is a structural understanding, not full code comprehension.

```
- Read the root directory listing
- Read package.json / Cargo.toml / pyproject.toml / go.mod or equivalent
- Read any existing README.md
- List top-level directories and infer their roles
- Identify entry points (main files, index files, server bootstraps)
- Identify configuration files (.env.example, config/, infrastructure/)
- Identify test structure (where tests live, what framework is used)
- Identify CI/CD setup (.github/workflows, Dockerfile, etc.)
```

Do **not** read every source file. Read only enough to understand boundaries and responsibilities.

---

### Step 2 — Identify system boundaries

Map the high-level components and how they relate. Look for:

- Frontend / backend separation
- API layer (REST, GraphQL, tRPC, etc.)
- Database access patterns (ORM, raw queries, repository pattern)
- Background jobs or workers
- External service integrations (third-party APIs, queues, storage)
- Shared libraries or internal packages
- Authentication and authorization mechanisms

For each boundary identified, note:
1. What it is responsible for
2. What it depends on
3. What depends on it

---

### Step 3 — Identify sensitive areas

Flag files, modules, or patterns that carry high system impact. These will be used by the PR Review Agent to enforce human review gates.

Default sensitive areas to look for:
- Authentication / session management
- Payment processing
- Database migrations
- Environment configuration and secrets handling
- Public API contracts (versioned routes, webhook handlers)
- Authorization / permission checks
- Infrastructure as code

Add project-specific ones based on what you find.

---

### Step 4 — Capture conventions

Read a small sample of source files (3–5 representative files across different layers) to capture:

- Naming conventions (files, functions, classes, variables)
- Error handling patterns
- Logging approach
- Testing patterns (unit vs integration, mocking strategy)
- Any explicit architectural patterns in use (CQRS, event sourcing, clean architecture layers, etc.)

Do not invent conventions that aren't there. If the codebase is inconsistent, note that explicitly.

---

### Step 5 — Decide what gets a reference file

Before writing anything, decide which domains are complex enough to warrant their own reference file. The rule is simple: if a domain needs more than 8–10 lines to describe properly, it gets its own file. If it can be captured in 2–3 lines, it stays in the index.

**Always create a reference file for:**
- Authentication & authorization (flows, token lifecycle, permission model)
- Payment processing (providers, flow, failure handling, webhook contracts)
- Data layer (schema overview, migration strategy, key relationships, query patterns)
- Any background job or event-driven system with non-trivial logic
- Any external integration that has its own failure modes or retry logic

**Keep in the index if:**
- The component has a single clear responsibility and no notable edge cases
- It is a thin wrapper around a well-known library
- It has no downstream consumers or upstream dependencies worth documenting

Create the list of reference files to generate before writing a single one. This is the reference manifest — it becomes the `Reference Docs` section of `ARCHITECTURE.md`.

---

### Step 6 — Write reference files

For each domain identified in Step 5, create `docs/arch/[domain].md`.

Each reference file follows this structure:

```markdown
# [Domain Name]

> Part of: ARCHITECTURE.md
> Last updated: [date]

## Overview

[2–3 sentences on what this domain is responsible for and why it's complex enough to warrant its own doc]

## How It Works

[The actual detail — flows, sequences, key decisions, edge cases. Use diagrams in text/ascii if helpful. 
This is where depth lives. No length limit but no padding either.]

## Key Files & Entry Points

[The specific files an agent should read first when working in this domain]
| File | Role |
|------|------|

## Contracts & Interfaces

[APIs, events, or data shapes this domain exposes or depends on — the things an agent must not break]

## Sensitive Patterns

[Specific things to be careful about in this domain — beyond the global sensitive areas list]

## Common Change Patterns

[How changes in this domain typically look — what files usually change together, what tests to run, 
what to watch out for. This is the senior developer knowledge transfer section.]

## Open Questions

[Ambiguities specific to this domain]
```

---

### Step 7 — Write ARCHITECTURE.md at skill/architect/ARCHITECTURE.md

Use the structure below. Be concise. Every section should add information an agent couldn't infer from reading a single file. Omit sections that don't apply rather than leaving them empty.
The index is intentionally shallow. Any domain that needs depth has a reference file — link to it, don't inline it.

```markdown
# Architecture

> Last updated: [date]
> Generated by: Architecture Mapping Agent
> Codebase: [repo name or path]

## Stack

[Language, runtime, key frameworks — one line each with version if determinable]

## System Overview

[2–4 sentences describing what the system does and its primary components at the highest level]

## Component Map

[For each major component:]
### [Component Name]
- **Role**: What it's responsible for
- **Location**: Directory path(s)
- **Depends on**: Other components or external services it calls
- **Exposes**: APIs, events, or interfaces consumed by others

## Entry Points

[List primary entry points with file paths — server start, CLI commands, background job runners, etc.]

## Data Layer

- **Database(s)**: [type, ORM if any]
- **Migration strategy**: [how schema changes are managed]
- **Key models/entities**: [list with one-line descriptions]

## External Integrations

[List third-party services, APIs, or queues the system integrates with and which component owns each]

## Authentication & Authorization

[How auth works — token type, session strategy, permission model]

## Sensitive Areas

[List of files, directories, or patterns that require human review on change]
| Area | Why sensitive | Location |
|------|--------------|----------|

## Conventions

- **Naming**: [file, function, class conventions]
- **Error handling**: [pattern used]
- **Logging**: [approach]
- **Testing**: [framework, where tests live, what level of coverage is expected]

## Known Inconsistencies or Tech Debt

[Honest notes on areas where the codebase deviates from its own patterns — useful for agents to know before touching these areas]

## Reference Docs

[Generated reference files for complex domains. Agents: read the relevant file(s) before making decisions about that domain.]

| Domain | File | When to read |
|--------|------|--------------|
| [e.g. Auth & Sessions] | docs/arch/auth.md | Any ticket touching login, tokens, permissions, middleware |
| [e.g. Payment Flow] | docs/arch/payments.md | Any ticket touching billing, subscriptions, webhooks |
| [e.g. Data Layer] | docs/arch/data.md | Any ticket touching schema, migrations, queries |

## Open Questions

[Anything ambiguous that a human should clarify before agents rely on this document]
```

---

### Step 8 — Flag ambiguities

Before finishing, explicitly list anything that was unclear during exploration:

- Ambiguous directory ownership
- Multiple conflicting patterns for the same concern
- Missing documentation for a component that appears important
- Configuration that suggests features not visible in the code

Post these as **Open Questions** in the `ARCHITECTURE.md` so a human can answer them before the document is used by other agents.

---

### Step 9 — Generate the Architect Agent skill

Using everything discovered in this run, generate a project-specific Architect Agent skill at `skills/architect/SKILL.md`.

This is not a generic template. It must have the actual project domains, actual sensitive area paths, and actual reference file paths baked in. An agent running this skill on a ticket should never have to figure out what domains exist or which reference files are available — that work was done here.

The generated skill must follow this structure:

```markdown
---
name: architect-agent
version: 1.0.0
description: This skill is used to analyze an issue against the current architecture and generate a technical details subtask in the configured tracker.
---
# Architect Agent — [Project Name]

## Purpose

Per-ticket skill. Invoked after the Requirement Agent has written the functional
requirement on a tracker issue. Reads the issue, consults architecture docs, decides
if decomposition is needed, maps technical impact areas, creates a Technical Details
subtask, and moves the issue status.

## Runtime Configuration

- Load `config.md` before starting.
- Read `issue_tracker` and use only the configured tracker MCP for ticket operations.
- Use the MCP mapped to `issue_tracker` in `config.md`.
- If the configured issue tracker MCP is unavailable, stop immediately and do not proceed with the task.
- For every created subtask/comment/status update, include: `Skill-Version: architect-agent@1.0.0`.

---

## Inputs

- Parent issue ID
- `ARCHITECTURE.md`
- Relevant `docs/arch/*.md` files (listed below — load only what applies to the ticket)

**Do not read source files.** All codebase knowledge comes from architecture docs.
If docs are insufficient, flag an open question — do not explore the codebase.

---

## Available Reference Files

[Populated from the reference manifest built in Step 5. List every generated reference file with its "when to read" signal so the agent can make the load decision without opening ARCHITECTURE.md first.]

| File | Load when the ticket involves... |
|------|----------------------------------|
| docs/arch/[domain].md | [specific trigger — e.g. login, tokens, session, middleware] |
| docs/arch/[domain].md | [specific trigger] |

---

## Sensitive Areas — Human Review Gates

[Populated directly from the Sensitive Areas table in ARCHITECTURE.md. 
These are the exact paths and patterns the PR Review Agent will also use.]

| Area | Location | Flag if ticket touches... |
|------|----------|--------------------------|
| [area name] | [path] | [trigger description] |

---

## Procedure

### Step 0 — Resolve tracker from config

Read `config.md`, set issue tracker context, and verify the configured tracker MCP is available before any ticket operation.

### Step 1 — Read the issue

Read the issue: title, description, existing comments, labels.
Extract the core intent before proceeding. Return to it if analysis drifts.

### Step 2 — Load architecture context

Always load `ARCHITECTURE.md` first. Then use the Available Reference Files table above
to decide which domain files to load. Load only what is relevant to this ticket.

Log every reference file referred to make the decisions — this goes into the Technical Details subtask verbatim.

### Step 3 — Decide: decompose or proceed

Decompose if the requirement touches more than 2 unrelated domains, or if parts can
ship and be validated independently.

Do not decompose if sub-tasks must ship together, or if the work is contained within
one domain even across multiple files.

If you decide decomposition is needed, do **not** create child issues in the tracker. Instead, 
add a recommendation to decompose in the Technical Details subtask (Step 5) with the rationale.

Continue to Step 4 regardless of the decomposition decision.

### Step 4 — Map impact areas

Identify from the architecture docs:

- **Candidate modules**: components and paths most likely to require changes
- **Blast radius**: components not changing but potentially affected
- **Sensitive area flags**: cross-reference against the Sensitive Areas table above — list matches explicitly, or state "None" explicitly
- **Reference files for Pre-Implementation Agent**: exact list of docs/arch/*.md files it referred to make above decisions

### Step 5 — Create Technical Details subtask

Create a subtask on the parent issue titled **"Technical Details"** with this body:

~~~markdown
## Technical Details

> Created by: Architect Agent
> Skill-Version: architect-agent@1.0.0
> Refs loaded: [every architecture file consulted in this run]

### Scope Decision
[One sentence: proceed as-is OR recommendation to decompose with rationale]

### Candidate Modules
- `path/to/module` — reason

### Blast Radius
- `path/to/component` — how it may be affected

### Sensitive Area Flags
[List matches from the sensitive areas table, or state: "None — this ticket does not touch any sensitive areas"]

### Reference Files for Pre-Implementation Agent
- `docs/arch/[domain].md` — why relevant

### Open Questions
[Questions that need human input before Pre-Implementation Agent runs. Leave empty if none.]
~~~

### Step 6 — Update issue status

- No open questions → set status to `ready-for-pre-imp`
- Open questions exist → set status to `needs-clarification`, comment on the issue
  tagging the relevant human with the questions. Do not move forward until resolved.

---

## What This Agent Does Not Do

- Does not read source files — only architecture docs
- Does not assign story points — that is the Pre-Implementation Agent's responsibility
- Does not create implementation subtasks — that is the Pre-Implementation Agent's responsibility
- Does not make code decisions — only scoping and impact mapping
```

---

## Token Efficiency Rules

This skill is the most expensive in the pipeline because it reads the codebase. Apply these rules strictly to keep it bounded:

1. **Never read files you don't need.** Use directory listings and file names to infer before opening.
2. **Read config and manifest files first.** They give the most information per token.
3. **Sample, don't scan.** Read 3–5 representative files per layer, not all of them.
4. **Stop when you have enough.** The goal is a useful map, not perfect coverage.
5. **If uncertain, ask.** Flag the ambiguity in Open Questions rather than reading more files to resolve it.
6. **Depth belongs in reference files, not the index.** If you're writing more than 8 lines about a domain in `ARCHITECTURE.md`, stop and move it to a reference file instead.
7. **Reference files are read on demand.** Downstream agents load only what's relevant to the ticket. A well-structured reference file that's only loaded 20% of the time is cheaper than stuffing that content into the index that's loaded 100% of the time.

---

## Quality Bar

The output is good if:

- The Architect Agent can make a decomposition and scoping decision after reading `ARCHITECTURE.md` alone for simple tickets, or `ARCHITECTURE.md` + one reference file for domain-specific tickets
- The Pre-Implementation Agent can identify candidate files without a full codebase scan
- The PR Review Agent knows exactly which files trigger a human review gate
- A new developer would understand the system's structure in under 5 minutes of reading the index
- No section in `ARCHITECTURE.md` exceeds 8–10 lines — anything longer was moved to a reference file
- The generated `.claude/skills/architect/SKILL.md` contains no generic placeholders — every domain, path, and sensitive area is specific to this project

It is not good if:
- The index contains detail that belongs in a reference file
- Reference files exist but aren't linked from the index with a clear "when to read" signal
- The generated Architect Agent skill has placeholders a human needs to fill in after the fact
- It describes what code does rather than how the system is structured
- It contains sections that are empty or say "N/A" — those should be omitted
- It makes assumptions without flagging them as assumptions
