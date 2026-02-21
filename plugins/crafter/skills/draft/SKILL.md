---
name: draft
description: Plan phase of RPI methodology. Consumes research artifact or topic and produces compact implementation plan with test specs and Agent Context blocks. Use after research or to start planning a non-trivial feature.
triggers:
  - "plan"
  - "draft plan"
  - "implementation plan"
allowed-tools: Read Glob Write Bash EnterPlanMode ExitPlanMode AskUserQuestion
---

# Craft Plan Skill

**RPI Phase 2 of 3:** Research → **Plan** → Implement

Use this skill after research (or standalone) to create a compact implementation plan with behavioral test specifications and Agent Context blocks for each phase.

## Purpose

The Plan phase creates a compact spec that fits in context by:
- Defining files to create/modify in order of operations
- Specifying test specs as behavioral descriptions at architectural boundaries
- Including Agent Context blocks so `/craft` can dispatch isolated agents per phase
- Setting clear acceptance criteria
- Identifying phase boundaries for incremental implementation

**Output:** Compact plan (~200 lines) at `docs/plans/YYYY-MM-DD-{topic}-plan.md`

## When to Use

Use this skill when:
- Completing research phase (has research artifact)
- Starting a non-trivial feature (no research artifact, will create one)
- Need a clear implementation roadmap before coding
- Want to specify test boundaries before implementing

**Don't use** for:
- Simple bug fixes
- Trivial features with obvious implementation
- Exploratory spikes

## Workflow

### 1. Consume Research Artifact (If Exists) — BEFORE Plan Mode

If research artifact exists:
```
Read: docs/plans/YYYY-MM-DD-{topic}-research.md
```

If no research artifact, create an inline summary (condensed):
- Identify relevant files
- Note existing patterns
- List integration points

### 2. Define Plan Scope — BEFORE Plan Mode

Ask the user to clarify if needed:
- What are the acceptance criteria?
- Are there specific constraints or preferences?
- What's the priority (MVP vs full feature)?

### 3. Enter Plan Mode

Call `EnterPlanMode` to switch to plan mode. This restricts available tools to read-only operations plus writing to the plan file — appropriate for planning work.

### 4. Create Implementation Plan — IN Plan Mode

Write the plan to the plan file using the [template structure](references/template.md).

**Required sections:**
- **Goal** — 1-2 sentence description of what we're building and why
- **Acceptance Criteria** — Testable outcomes as checklist
- **Files to Create** — Organized by layer (Core, Features, Shell, Tests)
- **Files to Modify** — What changes in existing files
- **Implementation Phases** — Ordered steps, each with:
  - Clear goal
  - Behavioral test specification (what to test, not how)
  - Tasks list
  - Verification checklist
  - **Agent Context block** (file paths, test command, gates, constraints)
- **Constraints & Considerations** — Architectural, testing, performance, security
- **Out of Scope** — Explicitly deferred features
- **Approval Checklist** — Pre-implementation verification
- **Next Steps** — Path to `/craft`

**Phase ordering:**
1. Database Schema (if needed)
2. Core Logic (L3 boundary — property/invariant tests)
3. Repository Layer (if needed)
4. Feature Use Cases (L3 boundary — behavioral assertions)
5. HTTP Routes (L4 boundary — contract tests)
6. Full Integration (verification)

See [template.md](references/template.md) for the complete template with Agent Context block reference.

### 5. Exit Plan Mode — User Reviews

Call `ExitPlanMode` to present the plan for user review and approval.

### 6. Persist Artifact — AFTER Approval

After the user approves the plan, save it to version control:
```
docs/plans/YYYY-MM-DD-{topic}-plan.md
```

Use kebab-case for the topic slug (e.g., `add-discount-codes-plan.md`).

### 7. Prompt Next Steps

```
Implementation plan saved. Next steps:
- Run `/craft` to execute this plan phase by phase
- Each phase will be executed by isolated agents (test writer → implementer → validator)
- If changes needed, clarify what to adjust and I'll update the plan
```

## Test Specifications at Boundaries

Test specs in the plan MUST be behavioral descriptions, not tool-specific code. The agents executing `/craft` will consult the project's CLAUDE.md for testing tools and patterns.

### L3 Core Tests (Property/Invariant)

Describe invariant properties that must hold:
- **Good:** "Splitting a total into N parts preserves the total"
- **Bad:** Code snippet using a specific testing library

### L3 Feature Tests (Behavioral)

Describe behavior at the use case boundary:
- **Good:** "Creating an order returns success with an order ID and correct total"
- **Bad:** `expect(mockRepository.create).toHaveBeenCalledWith(...)`

### L4 HTTP Tests (Contract)

Describe the HTTP contract:
- **Good:** "POST /orders returns 201 with `{ id, total }` on success, 400 with `{ error }` on validation failure"
- **Bad:** Testing internal handler implementation details

See [template.md](references/template.md) for detailed guidelines.

## Agent Context Blocks

Every implementation phase with tests MUST include an `#### Agent Context` subsection. This is the contract between `/draft` and `/craft` — it provides everything an isolated agent needs.

**Required fields:**
- **Files to create/modify** — explicit paths
- **Test spec** — behavioral description (what to test)
- **Test command** — shell command to run
- **RED gate / GREEN gate** — observable success criteria
- **Architectural constraints** — boundaries the agent must respect

See [template.md](references/template.md) for the Agent Context block reference.

## Phase Boundaries

Each phase should:
1. **Have a clear goal** — What capability is being added?
2. **Be independently verifiable** — Can you confirm it works?
3. **Be in logical order** — Database → Core → Features → Routes
4. **Have a self-contained Agent Context** — An isolated agent can execute it

Good boundaries: Database schema, Core functions, Repository, Feature use cases, HTTP routes

Bad boundaries: "Implement everything", mixing multiple layers

## Anti-Patterns to Avoid

- **Don't write full implementations** — Specs only, not code
- **Don't skip Agent Context blocks** — Each phase needs one for `/craft` to dispatch agents
- **Don't prescribe testing tools** — Describe behavior, reference project CLAUDE.md for tools
- **Don't create giant phases** — Keep phases small and focused
- **Don't omit verification steps** — Each phase needs concrete, observable checks

## After Planning

Once plan is complete and reviewed:
1. User reviews plan (via `ExitPlanMode`)
2. User approves or requests changes
3. Plan persisted to `docs/plans/`
4. Run `/craft` with plan artifact as input
5. Implementation executes phase by phase with isolated agents

## Context Compaction

**Why plan after research?** The plan phase is a compaction point:
- **Before planning:** Research artifact (~200 lines of findings)
- **After planning:** Implementation spec (~200 lines of actionable phases with Agent Context)
- **Implementation phase** works from compact plan, not raw research

This prevents context thrashing and keeps implementation focused.
