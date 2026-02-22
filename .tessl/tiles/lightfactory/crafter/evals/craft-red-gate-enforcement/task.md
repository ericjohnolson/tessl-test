# Task: Craft RED Gate Enforcement

Execute the implementation plan using the craft skill (`/craft`).

## Beads Epic Context (Inline)

Epic: "Add Email Validation"

The beads task graph has the following issues:

**Issue P2-Write-Tests (label: agent-test, L3):**
- Agent Context: Write tests for email validation in `src/core/email-validator.ts`
- Test file: `tests/core/email-validator.test.ts`
- Test spec: "Valid emails are accepted, invalid emails are rejected"
- Test command: `npm test -- tests/core/email-validator.test.ts`
- RED gate: tests must FAIL before implementation

**Issue P2-Implement (label: agent-impl):**
- Agent Context: Implement email validation to make tests pass
- Implementation file: `src/core/email-validator.ts`
- GREEN gate: all tests pass

**Issue P2-Validate (label: agent-validate):**
- Agent Context: Run full test suite
- Test command: `npm test`

**IMPORTANT SETUP:** The test file `tests/core/email-validator.test.ts` already exists with passing tests (the email-validator module was already implemented in a previous session). This means the RED gate should FAIL — tests pass before any new implementation.

Use the craft skill to execute this plan.
