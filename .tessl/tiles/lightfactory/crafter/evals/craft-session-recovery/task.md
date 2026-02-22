# Task: Craft Session Recovery

Resume an interrupted implementation session using the craft skill (`/craft`).

## Beads Epic Context (Inline)

Epic: "Add Inventory Tracking"

This is a **partially completed** epic. Some issues are already closed (completed in a previous session). The craft skill should pick up where it left off.

**Issue P1 (label: no-test) — CLOSED:**
- Title: "P1: Apply Schema — add inventory table"
- Status: closed (completed)

**Issue P2-Write-Tests (label: agent-test, L3) — CLOSED:**
- Title: "P2: Write Tests — Inventory Core"
- Status: closed (completed)
- Test file exists on disk: `tests/core/inventory.test.ts`

**Issue P2-Implement (label: agent-impl) — CLOSED:**
- Title: "P2: Implement — Inventory Core"
- Status: closed (completed)
- Implementation exists: `src/core/inventory.ts`

**Issue P2-Validate (label: agent-validate) — CLOSED:**
- Title: "P2: Validate — Inventory Core"
- Status: closed (completed)

**Issue P3-Write-Tests (label: agent-test, L3) — OPEN (ready):**
- Title: "P3: Write Tests — Stock Check Use Case"
- Blocked by: P2-Validate (closed, so this is unblocked)
- Agent Context: Write tests for checking stock availability
- Test file: `tests/features/check-stock.test.ts`
- Test command: `npm test -- tests/features/check-stock.test.ts`
- RED gate: tests must fail

**Issue P3-Implement (label: agent-impl) — OPEN (blocked):**
- Title: "P3: Implement — Stock Check Use Case"
- Blocked by: P3-Write-Tests
- Agent Context: Implement `src/features/check-stock.ts` to pass tests
- GREEN gate: tests pass

**Issue P3-Validate (label: agent-validate) — OPEN (blocked):**
- Title: "P3: Validate — Stock Check Use Case"
- Blocked by: P3-Implement
- Agent Context: Run full test suite
- Test command: `npm test`

## Environment Note

**Beads is NOT available in this environment.** Use the inline execution mode — treat the epic context above as if returned by `beads:show`. The closed issues represent completed work. Start from the first OPEN (ready) issue.

Use the `/craft` skill immediately to resume this implementation.
