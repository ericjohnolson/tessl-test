# Task-Specific Guidance Patterns and Teaching Techniques

## Teaching Techniques

### Socratic Questioning

Best for understanding requirements, design decisions, and debugging approach.

- "What do you think happens when this function receives null?"
- "What are the trade-offs of approach A vs approach B?"
- "What would a test for this behavior look like?"

### Progressive Hints

Best for when the engineer is stuck. Escalate only if needed:

| Level | Example |
|-------|---------|
| **Nudge** | "Look at what the function signature expects vs what you're passing." |
| **Direction** | "The issue is in how you're handling the async response." |
| **Specific** | "Check line 42 — you're accessing `.data` but the response wraps it in `.body.data`." |
| **Detailed** (Collaborative only) | "Here's the pattern you need: [pseudocode or code]" |

### Think-Aloud Pairing

Best for complex problems and unfamiliar patterns. Explain reasoning step-by-step, then ask the engineer to implement.

## Task-Specific Guidance Patterns

**Feature Implementation**: User-visible behavior → component breakdown → build order → data needs → error handling → tests

**Bug Fixing**: Reproduce → expected behavior → hypothesis → verify before changing → minimal fix → prevention

**Code Review Learning**: Purpose of change → convention adherence → edge cases → testability → scalability

## Complexity Calibration

Combine detected complexity with stated familiarity:

- **New + High complexity**: Maximum scaffolding, fundamentals-first
- **Experienced + Low complexity**: Minimal guidance, focus on trade-offs
- **New + Low complexity**: Moderate guidance, good learning opportunity
- **Experienced + High complexity**: Think-aloud pairing, focus on architecture
