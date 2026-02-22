---
name: craft
description: Implements a feature by executing a plan phase-by-phase — writes tests first, runs them red, then implements code to make them green, then validates nothing is broken. Use when you have a plan from /draft and are ready to implement, build, develop, create, or write code for a feature using TDD.
allowed-tools: Read Glob Write Bash Task TaskOutput Skill
---

# Craft Implement Skill

**RPI Phase 3 of 3:** Research → Plan → **Implement**

Use this skill to execute an implementation plan using beads-driven orchestration with isolated agents for strict test-first discipline.

## Purpose

Executes a beads task graph created by `/draft`. Each beads issue is a self-contained agent task with everything needed for dispatch. The dependency graph enforces ordering; `beads:ready` drives execution.

**Input:** Beads epic with per-agent-step issues (created by `/draft`)
**Output:** Working feature with passing tests

## When to Use

Use this skill when:
- Have a complete beads task graph from `/draft`
- Ready to execute plan phase by phase
- Want strict test/implementation isolation enforced
- Resuming an interrupted implementation session

**Don't use** for:
- Exploratory coding without a plan
- Simple bug fixes
- Research or planning tasks

## Workflow

### 1. Identify the Epic

Find the target epic via user input or `beads:search`. Verify the epic has issues with dependencies wired.

If resuming a previous session, this step is the same — `beads:ready` will return only unblocked, uncompleted tasks.

### 2. Beads-Driven Orchestration Loop

This is the core execution loop. It runs until all issues in the epic are closed or an unrecoverable error occurs.

```
Loop:
  a. Run beads:ready for the epic → list of unblocked tasks
  b. If no ready tasks and open tasks remain → something is blocked, escalate to user
  c. If no ready tasks and no open tasks remain → all done, proceed to final verification
  d. For each ready task:
     - Read issue description (contains full Agent Context)
     - Determine agent type from label (agent-test, agent-impl, agent-validate, no-test)
     - Dispatch Task with agent prompt built from issue description
     - If multiple ready tasks: dispatch in parallel (single message, multiple Task calls)
  e. Wait for agent(s) to complete
  f. For each completed agent:
     - If gate PASSED → close the issue via beads:close → unblocks dependents
     - If RED gate FAILED (tests pass immediately) → STOP, report to user
     - If GREEN gate FAILED → proceed to validation anyway
     - If VALIDATE found failures → create remediation issues (see Remediation)
  g. Loop back to (a)
```

See [workflow-detail.md](references/workflow-detail.md) for agent prompt templates and dispatch details.

#### Dispatching Agents

For each ready issue, build the agent prompt from the issue description:

1. Read the issue description via `beads:show`
2. The description contains the full Agent Context — file paths, test specs, commands, gates, constraints
3. Dispatch via `Task` tool (synchronous, `subagent_type: general-purpose`)
4. The agent does NOT need to read the plan file — the issue is self-contained

#### Parallel Dispatch

When `beads:ready` returns multiple tasks, dispatch them all in a **single message with multiple `Task` tool calls**. This happens naturally when:
- Two independent phases have no dependency between them
- A no-test phase and a test-write phase are both unblocked

TDD phases can't parallelize internally (Implement needs Write Test's output on disk), but independent phases parallelize across each other automatically via the dependency graph.

#### Remediation

When Agent 3 (Validate) finds failures:

1. Create a remediation issue via `beads:create` with label `agent-remediate`:
   - Title: `P{N}: Remediate — {Phase Name} (attempt {M})`
   - Description: includes failure output from Agent 3 (see [workflow-detail.md](references/workflow-detail.md) for template)
   - Blocked-by the failed Validate issue
2. Create a re-validation issue via `beads:create` with label `agent-validate`:
   - Title: `P{N}: Re-Validate — {Phase Name} (attempt {M})`
   - Blocked-by the remediation issue
3. Wire the next phase's first issue to be blocked-by the re-validation issue (replacing the original Validate dependency)
4. Close the original Validate issue (it completed its job — reporting failures)
5. If attempt count reaches 2 and re-validation still fails, update the issue to `blocked` status and **STOP — ask the user**

### 3. Report Progress

After each issue closes, report status. Use `beads:list` to show overall progress:
```markdown
**Progress Update:**
- [closed] P1: Apply Schema
- [closed] P2: Write Tests — Core Logic
- [closed] P2: Implement — Core Logic
- [open]  P2: Validate — Core Logic (in progress)
- [open]  P3: Repository Layer (blocked)
- [open]  P4: Write Tests — Apply Discount (blocked)
```

### 4. Final Verification

After all issues in the epic are closed:
1. Run the full test suite one final time
2. Verify all acceptance criteria from the epic are met
3. Report the agent execution summary (issues closed, remediations)
4. Suggest next steps (commit, PR, follow-up)

## Agent Isolation Discipline

**CRITICAL:** The three-agent pattern maintains honest separation between tests and implementation.

### Agent Roles

1. **Agent 1 (Write Test):** Creates failing tests only — never writes implementation code
2. **Agent 2 (Implement):** Writes implementation only — never modifies test files
3. **Agent 3 (Validate):** Runs tests and reports only — modifies nothing
4. **Agent 2-R (Remediate):** Fixes implementation only — never modifies test files
5. **Each agent starts fresh** — no shared context; each reads from the beads issue and files on disk

### Gate Failures

**RED Gate failed** (Agent 1's tests pass immediately before implementation):
1. STOP — the test is tautological or the feature already exists
2. Report to user and leave the issue open for their decision

**GREEN Gate failed** (Agent 2 cannot make tests pass):
1. Proceed to Agent 3 anyway to capture the full failure report
2. Enter remediation via dynamic issue creation
3. After 2 failed remediations, STOP and ask user

### What Not to Do

- **Don't skip Agent 3** — validation catches regressions in other tests
- **Don't read the plan file during execution** — beads issues are self-contained
- **Don't improvise beyond the plan** — stick to the beads issues or update them explicitly
- **Don't run agents in background** — synchronous dispatch ensures ordering

## Session Recovery

Recovery is trivial: run `beads:ready` for the epic.

- **Closed issues** = completed work (agents ran, gates passed, files on disk)
- **Ready issues** = next tasks to dispatch
- **Blocked issues** = waiting on dependencies or human input

No special recovery logic needed. The beads state *is* the execution state.

## After Implementation

Once all issues closed and verified:
1. **Run full test suite** — confirm everything passes
2. **Verify acceptance criteria** — all must be met
3. **Manual smoke test** — try key user journeys if applicable
4. **Create commit** — use `/commit` skill
5. **Document follow-up** — note any deferred work

