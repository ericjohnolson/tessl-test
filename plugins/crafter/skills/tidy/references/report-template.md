# Tidy Report Template

## Structure

```markdown
# Tidy Report

**Date:** YYYY-MM-DD
**Project:** {project name}
**Status:** Audit Complete

## Summary

{2-3 sentence overview of findings across all three categories.}

**Findings:** {total} total — {N} must-fix, {N} should-fix, {N} nice-to-have

## CLAUDE.md Structure

### Finding: {short description}
**Severity:** must-fix | should-fix | nice-to-have
**Details:** {what's wrong and why it matters for agents}
**Suggested Action:** {specific fix}

### Finding: {short description}
...

## Stale References

### Finding: {short description}
**Severity:** must-fix | should-fix | nice-to-have
**File:** `{path/to/file.md}`
**Details:** {what's broken — e.g., link target doesn't exist, file was renamed}
**Suggested Action:** {update link to X, remove reference, etc.}

### Finding: {short description}
...

## Missing Context

### Finding: {short description}
**Severity:** must-fix | should-fix | nice-to-have
**Evidence:** {where the undocumented pattern exists in code}
**Details:** {what should be documented and why agents need it}
**Suggested Action:** {add section to CLAUDE.md, create new doc, etc.}

### Finding: {short description}
...

## No Issues Found

{If a category has no findings, state it explicitly:}
No issues found in this category.

## Notes

- {Any observations that don't rise to finding level}
- {Patterns that are borderline but not worth a fix}
```

## Severity Definitions

| Severity | Definition | Examples |
|----------|-----------|---------|
| **must-fix** | Actively misleading. An agent following this documentation will make incorrect decisions. | Wrong build command, reference to deleted architecture doc, incorrect file paths |
| **should-fix** | Missing or outdated. Won't cause wrong behavior but reduces agent effectiveness. | Missing test command, undocumented env var needed for setup, outdated architecture description |
| **nice-to-have** | Would improve agent experience. Absence isn't harmful. | Better section ordering, adding a conventions section, documenting optional env vars |

## Quality Standards

### Completeness
- [ ] All three categories audited
- [ ] Every finding has severity, details, and suggested action
- [ ] Categories with no issues explicitly state "No issues found"

### Actionability
- [ ] Each suggested action is specific enough to implement without ambiguity
- [ ] Findings reference exact file paths and line numbers where possible
- [ ] Must-fix items clearly explain what agents will get wrong

### Conciseness
- [ ] Findings are brief and scannable
- [ ] No redundant findings (deduplicated across categories)
- [ ] Report is under 200 lines
