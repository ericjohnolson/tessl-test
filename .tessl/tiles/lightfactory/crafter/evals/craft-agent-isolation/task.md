# Task: Craft Agent Isolation

Execute the implementation plan using the craft skill (`/craft`).

## Beads Epic Context (Inline)

Epic: "Add Currency Converter"

The beads task graph has the following issues with dependencies wired:

**Issue P1 (label: no-test):**
- Title: "P1: Apply Schema — add exchange_rates table"
- No blockers

**Issue P2-Write-Tests (label: agent-test, L3):**
- Title: "P2: Write Tests — Currency Conversion Core"
- Blocked by: P1
- Agent Context: Write property-based tests for currency conversion
- Test file: `tests/core/currency-converter.test.ts`
- Test command: `npm test -- tests/core/currency-converter.test.ts`
- RED gate: tests must fail

**Issue P2-Implement (label: agent-impl):**
- Title: "P2: Implement — Currency Conversion Core"
- Blocked by: P2-Write-Tests
- Agent Context: Implement `src/core/currency-converter.ts` to pass tests
- GREEN gate: tests pass

**Issue P2-Validate (label: agent-validate):**
- Title: "P2: Validate — Currency Conversion Core"
- Blocked by: P2-Implement
- Agent Context: Run full test suite
- Test command: `npm test`

## Environment Note

**Beads is NOT available in this environment.** Use the inline execution mode — treat the epic context above as if returned by `beads:show`. Process issues in dependency order using the inline context. Do NOT attempt `beads:search` or other `beads:*` commands.

Use the `/craft` skill immediately to execute this plan.
