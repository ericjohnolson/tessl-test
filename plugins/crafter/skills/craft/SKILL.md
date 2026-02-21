---
name: craft
description: Implement phase of RPI methodology. Loads plan and executes phase by phase using isolated agents for test-first discipline. Use when executing an implementation plan from the draft skill.
triggers:
  - "implement"
  - "execute plan"
  - "build from plan"
allowed-tools: Read Glob Write Bash Task TaskOutput
---

# Craft Implement Skill

**RPI Phase 3 of 3:** Research → Plan → **Implement**

Use this skill to execute an implementation plan phase by phase using isolated agents for strict test-first discipline.

## Purpose

The Implement phase executes a compact plan using three agents per phase:
- **Agent 1 (Write Test):** Creates failing tests from the phase's test spec — knows nothing about the implementation
- **Agent 2 (Implement):** Writes minimal code to make tests pass — guided only by the tests
- **Agent 3 (Validate):** Runs the full test suite — confirms nothing is broken

This isolation ensures tests are honest (not written to match implementation) and implementation is minimal (driven only by test expectations).

**Input:** Implementation plan at `docs/plans/YYYY-MM-DD-{topic}-plan.md`
**Output:** Working feature with passing tests

## When to Use

Use this skill when:
- Have a complete implementation plan from `/draft`
- Ready to execute plan phase by phase
- Want strict test/implementation isolation enforced
- Need incremental progress reporting

**Don't use** for:
- Exploratory coding without a plan
- Simple bug fixes
- Research or planning tasks

## Workflow

### 1. Load Implementation Plan

Read the plan artifact:
```
Read: docs/plans/YYYY-MM-DD-{topic}-plan.md
```

Extract:
- Implementation phases with Agent Context blocks
- Acceptance criteria
- Full test suite command

### 2. Execute Phases with Agent Dispatch

For each phase in the plan, dispatch three sequential agents. Use the prompt templates from [workflow-detail.md](references/workflow-detail.md).

**For phases WITH tests (L3/L4 boundaries):**

#### Step A: Dispatch Agent 1 (Write Test)

Dispatch via `Task` tool (synchronous, `subagent_type: general-purpose`):
- Pass the plan path and phase number
- Include the Agent Context block from the plan
- Agent writes test files and runs them to confirm failure

**Verify RED gate:** Agent 1 must report that tests fail for the expected reason. If tests pass immediately, STOP — do not proceed.

#### Step B: Dispatch Agent 2 (Implement)

Dispatch via `Task` tool (synchronous, `subagent_type: general-purpose`):
- Pass the plan path, phase number, and test file paths from Agent 1
- Agent writes implementation and runs tests to confirm they pass

**Verify GREEN gate:** Agent 2 must report all tests passing. If tests fail, proceed to remediation.

#### Step C: Dispatch Agent 3 (Validate)

Dispatch via `Task` tool (synchronous, `subagent_type: general-purpose`):
- Pass the full test suite command and file lists from Agents 1 and 2
- Agent runs the full suite and reports results

**Verify COMPLETE:** If all tests pass, proceed to next phase. If failures found, enter remediation loop.

**For phases WITHOUT tests (schema, infrastructure):**

Execute directly (no agent dispatch needed):
- Follow the tasks in the plan
- Verify the acceptance gate from the Agent Context block
- Proceed to next phase

#### Remediation Loop

If Agent 3 finds failures:
1. Dispatch Agent 2-R with failure output from Agent 3 (max 2 retries)
2. After each remediation, dispatch Agent 3 again to validate
3. If still failing after 2 remediation attempts, **STOP and ask the user**

### 3. Report Progress After Each Phase

After each phase completes, report status:
```markdown
**Progress Update:**
- ✅ Phase 1: Database Schema - Complete
- ✅ Phase 2: Core Logic - Complete (Agent 1→2→3)
- 🔵 Phase 3: Feature Layer - Agent 1 dispatched
- ⚪ Phase 4: HTTP Routes - Not Started
```

See [workflow-detail.md](references/workflow-detail.md) for progress templates.

### 4. Final Verification

After all phases complete:
1. Run the full test suite one final time
2. Verify all acceptance criteria from the plan are met
3. Report the agent execution summary (phases, remediations)
4. Suggest next steps (commit, PR, follow-up)

## Agent Isolation Discipline

**CRITICAL:** The three-agent pattern exists to maintain honest separation between tests and implementation.

### Rules

1. **Agent 1 writes tests only** — never implementation code
2. **Agent 2 writes implementation only** — never modifies test files
3. **Agent 3 modifies nothing** — only runs tests and reports
4. **Agent 2-R fixes implementation only** — never modifies test files
5. **Each agent starts fresh** — no shared context between agents

### If RED Gate Fails

If Agent 1's tests pass immediately (before implementation):
1. STOP — the test is tautological or the feature already exists
2. Report to user — explain what happened
3. Do NOT dispatch Agent 2

### If GREEN Gate Fails

If Agent 2 cannot make tests pass:
1. Proceed to Agent 3 anyway (to get full failure report)
2. Enter remediation loop with Agent 2-R
3. After 2 failed remediations, STOP and ask user

## Anti-Patterns to Avoid

- **Don't let agents share context** — each agent starts from the plan and files on disk
- **Don't skip Agent 3** — validation catches regressions in other tests
- **Don't modify tests during implementation** — tests define the contract
- **Don't skip phases** — execute in order, each depends on previous
- **Don't improvise beyond the plan** — stick to the plan or update it explicitly
- **Don't run agents in background** — synchronous dispatch ensures ordering

## After Implementation

Once all phases complete and verified:
1. **Run full test suite** — confirm everything passes
2. **Verify acceptance criteria** — all must be met
3. **Manual smoke test** — try key user journeys if applicable
4. **Create commit** — use `/commit` skill
5. **Document follow-up** — note any deferred work

## Context Compaction

**Why isolated agents?** Each agent loads only what it needs:
- **Agent 1:** Plan's test spec + project test patterns → writes tests
- **Agent 2:** Test files + plan's constraints → writes implementation
- **Agent 3:** Test command → runs and reports

No agent carries the full research or planning context. The plan's Agent Context blocks provide exactly the information each agent needs.
