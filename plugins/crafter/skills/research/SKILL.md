---
name: research
description: Research phase of RPI methodology. Spawns parallel subagents for codebase exploration AND web/pattern research, then synthesizes findings in plan mode for user review. Produces compact research artifact. Use at the start of non-trivial features.
triggers:
  - "research"
  - "explore codebase"
  - "investigate"
allowed-tools: Read Glob Grep Bash Task TaskOutput WebSearch WebFetch AskUserQuestion Write EnterPlanMode ExitPlanMode
---

# Research Skill

**RPI Phase 1 of 3:** Research → Plan → Implement

Use this skill at the start of non-trivial features to explore the codebase AND research external patterns before planning or implementing.

## Purpose

The Research phase prevents thrashing and discovers constraints early by:
- Exploring relevant parts of the codebase in parallel
- Researching external patterns, libraries, and best practices
- Identifying existing patterns and architectural boundaries
- Understanding dependencies and integration points
- Uncovering constraints before they become blockers

**Output:** Compact research artifact (~200 lines) at `docs/plans/YYYY-MM-DD-{topic}-research.md`

## When to Use

Use this skill when:
- Starting work on a non-trivial feature
- Need to understand how a new feature fits into existing architecture
- Want to discover existing patterns before creating new ones
- Need to identify integration points and dependencies
- Working in an unfamiliar part of the codebase
- Need to evaluate external libraries or architectural approaches

**Don't use** for:
- Simple bug fixes
- Trivial features with obvious implementation
- Pure research tasks (use Explore agent instead)

## Depth Levels

| Depth | Codebase Agents | Web Agents | When to Use |
|-------|----------------|------------|-------------|
| Quick | 2-3 | 0-1 | Well-understood pattern, single codebase area, no external unknowns |
| Standard | 3-4 | 1-2 | Multiple codebase areas affected, some external patterns to validate |
| Deep | 4-5 | 2-3 | Unfamiliar domain, new technology, multiple integration points |

**Choosing depth:** Infer the appropriate depth from the user's prompt using these signals:

| Signal | Points toward |
|--------|--------------|
| User names a specific file or function to change | Quick |
| Feature touches one module with known patterns | Quick |
| Feature spans multiple modules or layers | Standard |
| User mentions a library, API, or pattern they haven't used before | Standard or Deep |
| Feature involves new technology, unfamiliar domain, or architectural change | Deep |
| User explicitly says "quick look" or "just check" | Quick |
| User explicitly says "deep dive" or "thorough" | Deep |

**Not all research needs web agents.** If the feature is purely internal (no new libraries, APIs, or unfamiliar patterns), skip web research entirely and dispatch only codebase agents.

**When uncertain**, ask the user with `AskUserQuestion` — briefly explain what you'd investigate at each level and let them choose. Do NOT default to a deeper level than the prompt warrants.

## Workflow

### 1. Define Research Scope — BEFORE Plan Mode

Ask the user to clarify the research scope if needed:
- What is the feature or capability being built?
- Which architectural boundaries are likely affected?
- Are there specific concerns or constraints?

Based on the chosen depth, decompose the scope into:
- **Codebase investigation areas** (2-5 depending on depth)
- **Web research topics** (0-3 depending on depth)

### 2. Dispatch Parallel Agents — BEFORE Plan Mode

**CRITICAL: Dispatch ALL agents in a SINGLE message using multiple Task tool calls with `run_in_background: true`.**

Agents need the Task tool with full access, so this step MUST happen before plan mode.

Two agent types — see [agent prompts](references/agent-prompts.md) for full templates:

**Codebase Explorer** (subagent_type: `Explore`)
- Receives one investigation area
- Uses Read/Glob/Grep to examine up to 10 files
- Reports: files examined, patterns observed, architectural notes, relevance, gaps

**Web Researcher** (subagent_type: `general-purpose`)
- Receives one research topic + feature context
- Uses WebSearch/WebFetch to consult 2-4 sources
- Reports findings with **High/Medium/Low** confidence levels

Example dispatch for "Add discount codes to orders":

```
# ALL in a single message:
Task(Explore): "Investigate existing discount logic in core/ and features/"
Task(Explore): "Investigate order creation flow end-to-end"
Task(Explore): "Investigate validation patterns for codes/slugs"
Task(Explore): "Investigate database schema for orders and related tables"
Task(general-purpose): "Research discount/coupon code validation patterns and best practices"
Task(general-purpose): "Research Stripe coupon API integration patterns"
```

### 3. Collect Agent Results — BEFORE Plan Mode

- Poll agents with `TaskOutput block: false` to check progress
- Collect completed results with `TaskOutput block: true`
- If an agent returns thin results, note the gap — do NOT dispatch a follow-up agent

### 4. Enter Plan Mode

Call `EnterPlanMode` to switch to plan mode. Synthesis and artifact writing happen in plan mode because they are read/write-only activities — no tool dispatch needed.

### 5. Synthesize and Write Artifact — IN Plan Mode

Cross-reference codebase patterns against web findings:
- Align codebase patterns with web best practices
- Surface contradictions as open questions
- Discard irrelevant web findings that don't apply to the codebase context
- Low-confidence web findings become **Open Questions** unless corroborated by codebase evidence
- Flag contested findings where web sources disagree

Write the research artifact to the plan file using the [research artifact template](references/template.md). Target ~200 lines.

### 6. Exit Plan Mode — User Reviews

Call `ExitPlanMode` to present the research artifact for user review and approval.

### 7. Persist Artifact — AFTER Approval

After the user approves, save the artifact to version control:
```
docs/plans/YYYY-MM-DD-{topic}-research.md
```

Use kebab-case for the topic slug.

### 8. Prompt Next Steps

Present a brief summary of key findings, then prompt user to:
- Clarify open questions
- Proceed to `/draft` when satisfied

## Anti-Patterns to Avoid

- **Don't copy entire files** — note file paths and purpose only
- **Don't write code yet** — this is research, not implementation
- **Don't create plans** — that's the next phase (`/draft`)
- **Don't research sequentially** — use parallel agents dispatched in a single message
- **Don't include irrelevant details** — stay focused on the feature
- **Don't present web findings without confidence levels** — every web finding needs High/Medium/Low
- **Don't trust a single web source** — cross-reference when possible
- **Don't dispatch agents inside plan mode** — agents need full tool access; dispatch them before entering plan mode

## After Research

Once research is complete and reviewed:
1. User reviews findings (via `ExitPlanMode`)
2. User clarifies any open questions
3. Artifact persisted to `docs/plans/`
4. Run `/draft` with research artifact as input
5. Research artifact serves as context for planning

## Context Compaction

**Why research first?** The research phase is a compaction point:
- **Before research:** Entire codebase + unbounded web knowledge (too much context)
- **After research:** Compact artifact (~200 lines, only relevant details from both sources)
- **Planning phase** works from compact artifact, not raw codebase or web searches

This prevents context thrashing and keeps planning focused.
