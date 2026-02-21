# Craft Implement: Workflow Details

This reference provides detailed templates, examples, and procedures for executing the implementation workflow.

## Agent Dispatch Templates

Each implementation phase is executed by three sequential agents dispatched via the `Task` tool. Each agent operates in isolation — it does not share context with the other agents. This separation ensures the test-writer doesn't "know" the implementation and vice versa.

### Agent 1: Write Test

**Purpose:** Create failing tests for the current phase.
**Agent type:** `general-purpose` (subagent_type)
**Mode:** Synchronous (not background)

```
You are implementing Phase {N}: {Phase Name} of the plan at {plan_path}.

YOUR ROLE: Write failing tests ONLY. Do NOT write any implementation code.

## Context

Read the implementation plan at {plan_path}, specifically the Agent Context block for Phase {N}.

Key information from the Agent Context:
- Files to create: {test file paths from Agent Context}
- Test spec: {behavioral test description from Agent Context}
- Test command: {test command from Agent Context}
- RED gate: {what failure should look like from Agent Context}

## Instructions

1. Read the project's CLAUDE.md to understand testing tools, patterns, and conventions
2. Read any existing test files in the project to match style and patterns
3. Write test file(s) at the paths specified in the Agent Context
4. Tests MUST assert behavior at architectural boundaries — no internal mocks
5. Run the test command: {test command}
6. Verify tests FAIL for the expected reason (the RED gate)

## Report

After completing, report:
- **Test files created:** {list of file paths}
- **Test command output:** {paste the failure output}
- **RED gate status:** PASS (tests fail as expected) or FAIL (tests don't fail — explain why)

IMPORTANT: If tests pass immediately, something is wrong. Report this as a RED gate FAIL.
```

### Agent 2: Implement

**Purpose:** Write minimal code to make tests pass.
**Agent type:** `general-purpose` (subagent_type)
**Mode:** Synchronous (not background)

```
You are implementing Phase {N}: {Phase Name} of the plan at {plan_path}.

YOUR ROLE: Write minimal implementation to make existing tests pass. Do NOT modify test files.

## Context

Read the implementation plan at {plan_path}, specifically the Agent Context block for Phase {N}.

Key information from the Agent Context:
- Files to create/modify: {implementation file paths from Agent Context}
- Test command: {test command from Agent Context}
- GREEN gate: {what success should look like from Agent Context}
- Architectural constraints: {constraints from Agent Context}

The following test files were created by a previous agent:
{list of test file paths from Agent 1's report}

## Instructions

1. Read the test files to understand what behavior is expected
2. Read the project's CLAUDE.md to understand coding patterns and conventions
3. Read any existing source files that tests import or reference
4. Write the minimal implementation to make all tests pass
5. Respect the architectural constraints from the Agent Context
6. Run the test command: {test command}
7. Verify all tests PASS (the GREEN gate)

## Report

After completing, report:
- **Implementation files created/modified:** {list of file paths}
- **Test command output:** {paste the output}
- **GREEN gate status:** PASS (all tests pass) or FAIL (some tests fail — include failure output)

IMPORTANT: Do NOT modify test files. If a test seems wrong, report it and let a human decide.
```

### Agent 3: Validate

**Purpose:** Run the full test suite to ensure nothing is broken.
**Agent type:** `general-purpose` (subagent_type)
**Mode:** Synchronous (not background)

```
You are validating Phase {N}: {Phase Name} of the plan at {plan_path}.

YOUR ROLE: Run the full test suite and report results. Do NOT modify any files.

## Context

Phase {N} has been implemented. The following files were created/modified:
- Test files: {list from Agent 1}
- Implementation files: {list from Agent 2}

## Instructions

1. Run the full test suite: {full test suite command from plan}
2. Check for any failures — both in the new tests and in pre-existing tests
3. Report the results

## Report

- **Test command:** {command run}
- **Result:** ALL PASS or FAILURES FOUND
- **Total tests:** {count}
- **Failures:** {count and details if any}
- **Failure output:** {paste relevant failure output if any}

IMPORTANT: Do NOT fix anything. Only report.
```

### Agent 2-R: Remediation

**Purpose:** Fix implementation after Agent 3 finds failures, without modifying tests.
**Agent type:** `general-purpose` (subagent_type)
**Mode:** Synchronous (not background)

```
You are remediating Phase {N}: {Phase Name} of the plan at {plan_path}.

YOUR ROLE: Fix the implementation to make all tests pass. Do NOT modify test files.

## Context

Read the implementation plan at {plan_path}, specifically the Agent Context block for Phase {N}.

The validation agent found failures after implementation:

{paste Agent 3's failure output here}

Files involved:
- Test files (DO NOT MODIFY): {list from Agent 1}
- Implementation files (you may modify these): {list from Agent 2}

## Instructions

1. Read the failing tests to understand expected behavior
2. Read the implementation files to find the issue
3. Fix the implementation — minimal changes only
4. Respect the architectural constraints from the Agent Context
5. Run the full test suite: {full test suite command}
6. Verify ALL tests pass

## Report

- **Files modified:** {list of changed files}
- **What was fixed:** {brief description of the issue and fix}
- **Test command output:** {paste the output}
- **Result:** ALL PASS or STILL FAILING (include details)

IMPORTANT: Do NOT modify test files. Tests define the contract.
```

---

## Phase Execution Pattern Template

Use this template for tracking each phase during execution:

```markdown
## Phase {N}: {Phase Name}

**Goal:** {Phase goal from plan}

**Status:** 🔴 Starting

### Agent 1: Write Test → RED

**Dispatched:** {timestamp}
**Test files:** {paths from agent report}
**RED gate:** {PASS/FAIL}

### Agent 2: Implement → GREEN

**Dispatched:** {timestamp}
**Implementation files:** {paths from agent report}
**GREEN gate:** {PASS/FAIL}

### Agent 3: Validate → COMPLETE

**Dispatched:** {timestamp}
**Result:** {ALL PASS / FAILURES FOUND}

### Phase {N} Complete ✅

**Verification checklist:**
- [ ] Tests written by isolated agent (no implementation knowledge)
- [ ] Implementation written by isolated agent (guided only by tests)
- [ ] Full test suite passes after phase
- [ ] Phase verification steps from plan completed

**Proceeding to Phase {N+1}**
```

---

## Progress Reporting Templates

Use these templates to keep the user informed throughout implementation:

### Phase Start

```markdown
## Phase 2: Core Logic

**Goal:** Implement pure business logic for discount calculation
**Status:** 🔴 Starting — dispatching Agent 1 (Write Test)
```

### Phase In Progress

```markdown
**Status:** 🔵 Agent 2 (Implement) dispatched — making tests pass
```

### Phase Complete

```markdown
**Status:** ✅ Complete

- Agent 1: Tests written and confirmed RED ✅
- Agent 2: Implementation passes tests ✅
- Agent 3: Full suite validated ✅
```

### Overall Progress

```markdown
**Progress Update:**
- ✅ Phase 1: Database Schema - Complete
- ✅ Phase 2: Core Logic - Complete (3 agents)
- 🔵 Phase 3: Repository Layer - Agent 2 in progress
- ⚪ Phase 4: Feature Use Case - Not Started
- ⚪ Phase 5: HTTP Routes - Not Started
- ⚪ Phase 6: Full Integration - Not Started

**Progress:** 2/6 phases complete (33%)
```

## Error Handling Procedures

### Remediation Loop

If Agent 3 finds failures:
1. **First attempt:** Dispatch Agent 2-R with failure output (max context for debugging)
2. **Second attempt:** If still failing, dispatch Agent 2-R again with updated failure output
3. **Stop:** After 2 remediation attempts, STOP and ask the human for guidance

### If Agent 1 RED Gate Fails

If tests pass immediately (before implementation):
1. **STOP** — the test is not testing new behavior
2. **Report to human** — explain that tests pass without implementation
3. **Do NOT proceed** to Agent 2 — the phase is broken

### If Phase Cannot Be Completed

If a phase cannot be completed after remediation:
1. **Document the issue** — What failed and why
2. **Ask for guidance** — Should we adjust the plan?
3. **Don't skip ahead** — Phases depend on previous phases
4. **Don't work around** — Fix the root cause

### If Plan is Incomplete

If the Agent Context block is missing details:
1. **Note the gap** — What's unclear?
2. **Ask for clarification** — Get specifics before proceeding
3. **Don't guess** — Assumptions cause rework
4. **Don't improvise** — Stick to the plan or update it

## Quality Standards Checklists

### Test Quality
- [ ] Tests written by Agent 1 without knowledge of implementation
- [ ] Tests assert at L3/L4 boundaries
- [ ] Tests verify behavior, not implementation calls
- [ ] No internal mocks — only boundary testing

### Implementation Quality
- [ ] Implementation written by Agent 2 guided only by test expectations
- [ ] Uses existing patterns from project CLAUDE.md
- [ ] Minimal code to pass tests
- [ ] Respects architectural constraints from Agent Context

### Phase Verification
- [ ] Each phase validated by Agent 3 before proceeding
- [ ] Full test suite passes (not just new tests)
- [ ] Remediation attempts tracked (max 2 per phase)

## Final Verification Template

Use this template after all phases complete:

```markdown
## Final Verification

### Full Test Suite
{Run full test suite command from plan}

**Result:** {✅ All passing / ❌ Failures found}

### Acceptance Criteria

From plan:
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

### Agent Execution Summary

| Phase | Agent 1 (RED) | Agent 2 (GREEN) | Agent 3 (VALIDATE) | Remediations |
|-------|--------------|-----------------|-------------------|--------------|
| 1     | ✅           | ✅              | ✅                | 0            |
| 2     | ✅           | ✅              | ✅                | 0            |
| ...   | ...          | ...             | ...               | ...          |

---

## Implementation Complete

All phases executed successfully:
- ✅ All tests passing
- ✅ All acceptance criteria met
- ✅ No test skips
- ✅ Code follows architectural patterns
- ✅ Test/implementation isolation maintained across all phases

**Next steps:**
- Run `/commit` to create commit
- Create PR if needed
- Document any follow-up work
```
