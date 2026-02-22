---
name: tidy
description: Audit and fix agent-facing documentation. Use when CLAUDE.md may be stale, docs don't match code, or undocumented conventions exist.
triggers:
  - "tidy"
  - "tidy first"
  - "audit docs"
  - "update claude.md"
  - "agent docs"
allowed-tools: Read Glob Grep Bash Task TaskOutput Write AskUserQuestion
---

# Tidy Skill

Inspired by Kent Beck's *Tidy First?* philosophy: make small structural improvements before behavioral changes. This skill targets the documentation layer that AI agents depend on — CLAUDE.md, cross-references, and implicit conventions.

## Purpose

AI agents perform better when their working context is accurate. Over time, agent-facing documentation drifts from reality as code evolves. This skill audits and fixes that drift by:

- Checking CLAUDE.md structure against best practices
- Verifying doc references point to real, current files
- Surfacing undocumented conventions that exist in code but not in docs

**Output:** Tidy report artifact at `docs/plans/YYYY-MM-DD-tidy-report.md`

## When to Use

Use this skill when:
- Starting work in a repo with an existing CLAUDE.md that may be stale
- After significant refactoring that moved or renamed files
- Onboarding to a new codebase and want to validate its docs
- Before a major feature to ensure the agent surface is accurate

**Don't use** for:
- Writing new documentation from scratch (use documentation-writer)
- Code-level refactoring (use `/refactor`)
- Initial project setup (write CLAUDE.md manually first)

## Workflow

### Phase 1: Audit

#### 1. Dispatch Parallel Audit Agents

**CRITICAL: Dispatch ALL 3 agents in a SINGLE message using multiple Task tool calls with `run_in_background: true`.**

Three agents run in parallel — see [agent prompts](references/agent-prompts.md) for full templates:

| Agent | Type | Investigates |
|-------|------|-------------|
| CLAUDE.md Auditor | `Explore` | Structure, sections, best practices, cross-references |
| Reference Checker | `Explore` | Broken links, stale paths, outdated code references |
| Context Scout | `Explore` | Undocumented env vars, naming conventions, architectural patterns |

Example dispatch:

```
# ALL in a single message:
Task(Explore): "Audit CLAUDE.md structure against best practices for [project]"
Task(Explore): "Check all markdown files for broken internal links and stale references in [project]"
Task(Explore): "Scout for undocumented conventions and patterns in [project]"
```

#### 2. Collect Agent Results

- Poll agents with `TaskOutput block: false` to check progress
- Collect completed results with `TaskOutput block: true`
- If an agent returns thin results, note the gap — do NOT dispatch follow-up agents

#### 3. Synthesize Findings

Cross-reference agent results and deduplicate. For each finding, assign:

| Severity | Definition |
|----------|-----------|
| **must-fix** | Actively misleading. Agent will make wrong decisions based on this. |
| **should-fix** | Missing or outdated but won't cause incorrect behavior. Reduces agent effectiveness. |
| **nice-to-have** | Would improve agent experience but absence isn't harmful. |

Write the tidy report using the [report template](references/report-template.md).

Save the report to: `docs/plans/YYYY-MM-DD-tidy-report.md`

#### 4. Present Report to User

Use `AskUserQuestion` to present a summary of findings by severity:

```
Found N findings:
- X must-fix (actively misleading)
- Y should-fix (outdated/missing)
- Z nice-to-have (improvements)

Which findings should I address?
```

Options:
- All findings
- Must-fix only
- Must-fix and should-fix
- Let me pick individually

If the user picks individually, list each finding with its severity and let them select via `AskUserQuestion` with `multiSelect: true`.

### Phase 2: Fix

#### 5. Apply Fixes

For each approved finding, in order of severity (must-fix first):

1. Read the relevant file(s)
2. Apply the fix using `Write` or `Edit`
3. Commit with message format: `tidy: <description>`

**One fix per commit.** This mirrors Beck's philosophy of keeping tidying commits separate and small.

Examples of commit messages:
- `tidy: add missing build commands to CLAUDE.md`
- `tidy: fix broken link to architecture.md in README`
- `tidy: document DATABASE_URL env var in CLAUDE.md`
- `tidy: remove reference to deleted utils/helpers.ts`

#### 6. Summary

After all fixes are applied, present a brief summary:
- Number of findings addressed
- List of commits made
- Any findings skipped and why

## Audit Categories

### Category 1: CLAUDE.md Structure

Checks against [CLAUDE.md best practices](references/claude-md-best-practices.md):

- **Required sections present:** Project overview, key commands, architecture summary, conventions, key files
- **Section ordering:** Most important context first
- **Length and focus:** Not too long (aim for scannable), not too sparse
- **Cross-references:** Links to architecture docs, ADRs, or other referenced files exist and resolve
- **Accuracy:** Commands listed actually work, paths mentioned exist

### Category 2: Stale References

Scans all markdown files in the project:

- **Internal links:** `[text](path/to/file.md)` where target doesn't exist
- **Code references:** Mentions of specific files, functions, or classes that have been moved, renamed, or deleted
- **Command examples:** Shell commands or scripts referenced that no longer work
- **Feature references:** Mentions of features or behaviors that have been removed

### Category 3: Missing Context

Explores the codebase for patterns that should be documented:

- **Environment variables:** Used in code (e.g., `process.env.X`, `os.environ`) but not listed in docs
- **Naming conventions:** Consistent patterns in code (e.g., `*Repository`, `*UseCase`, `*Controller`) with no documentation
- **Directory structure conventions:** Implicit organization (e.g., feature folders, layer separation) not explained
- **Key commands:** Build, test, lint, or deploy commands in package.json/Makefile/etc. not documented in CLAUDE.md
- **Architectural patterns:** Dependency injection, event sourcing, hexagonal architecture, etc. visible in code but not described

## Anti-Patterns to Avoid

- **Don't rewrite CLAUDE.md from scratch** — make targeted fixes to existing content
- **Don't add documentation for everything** — focus on what agents need to make correct decisions
- **Don't bundle multiple fixes in one commit** — one finding, one commit
- **Don't fix code** — this skill fixes documentation only
- **Don't invent conventions** — only document patterns that demonstrably exist in code
- **Don't expand scope** — if you discover code issues, note them but don't fix them

## After Tidy

Once the tidy report is addressed:
1. Agent-facing documentation is accurate and current
2. Subsequent `/research`, `/draft`, and `/craft` sessions work from reliable context
3. Run `/tidy` again periodically or after major refactors
