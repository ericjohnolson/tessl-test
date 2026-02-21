# Agent Prompt Templates

Prompt templates for the three audit agents dispatched during the tidy workflow. Use these as the basis for Task tool prompts, filling in the bracketed placeholders.

## Agent 1: CLAUDE.md Auditor

**Agent type:** `Explore` (subagent_type)

```
Audit the CLAUDE.md file in [{project_root}] against best practices for agent-facing documentation.

Your goal is to identify structural issues, missing sections, and inaccurate content that would cause an AI agent to make wrong decisions or miss important context.

Instructions:
- Read CLAUDE.md (and any CLAUDE.local.md if present)
- Read the actual project structure using Glob and Bash (ls)
- Compare documented commands against actual build/test/lint configuration (package.json, Makefile, etc.)
- Check all cross-references and links in CLAUDE.md — verify targets exist
- Evaluate against these required sections:
  1. Project overview ("What This Project Is")
  2. Key commands (build, test, lint, run)
  3. Architecture summary (layers, patterns, directory conventions)
  4. Conventions (naming, file organization, coding patterns)
  5. Key files (entry points, config files, important modules)

Report format:

### CLAUDE.md Audit

**Sections Present:**
- {section name}: present | missing | incomplete

**Structural Issues:**
- {issue}: {description and why it matters for agents}

**Stale Content:**
- {line or section}: {what's wrong, what it should say}

**Broken Cross-References:**
- `{link}` in CLAUDE.md → target {exists | missing | moved to X}

**Missing Content:**
- {what's missing and why an agent needs it}

**Overall Assessment:**
- {brief summary of CLAUDE.md health}
```

## Agent 2: Reference Checker

**Agent type:** `Explore` (subagent_type)

```
Scan all markdown files in [{project_root}] for broken internal links, stale code references, and outdated paths.

Your goal is to find every reference in documentation that points to something that no longer exists or has moved.

Instructions:
- Use Glob to find all *.md files in the project
- For each markdown file, extract internal links: [text](relative/path) and [text](./relative/path)
- Use Glob or Read to verify each link target exists
- Look for inline code references to specific files (e.g., `src/foo/bar.ts`) and verify they exist
- Check for references to commands, scripts, or executables and verify they're valid
- Check for mentions of features, modules, or components by name — flag any that seem to reference removed functionality

Report format:

### Reference Check

**Files Scanned:**
- {N} markdown files checked

**Broken Links:**
- `{source_file}`: [{link_text}]({target}) → target does not exist
  Suggested fix: {update to X | remove link | target may have moved to Y}

**Stale Code References:**
- `{source_file}`: mentions `{code_path}` — file does not exist
  Suggested fix: {update path | remove reference}

**Outdated Commands:**
- `{source_file}`: references `{command}` — {reason it's outdated}
  Suggested fix: {correct command}

**Clean References:**
- {N} links verified as valid

**Summary:**
- {N} broken links, {N} stale code refs, {N} outdated commands found
```

## Agent 3: Context Scout

**Agent type:** `Explore` (subagent_type)

```
Explore the codebase at [{project_root}] for patterns, conventions, and configuration that should be documented but aren't.

Your goal is to find implicit knowledge that an AI agent would need but can't discover from existing documentation alone.

Instructions:
- Search for environment variable usage (process.env, os.environ, env., System.getenv, etc.)
- Cross-reference found env vars against documentation — flag any not documented
- Analyze directory structure for naming conventions (e.g., consistent suffixes like *Repository, *Controller, *UseCase)
- Look at package.json scripts, Makefile targets, or similar for undocumented commands
- Examine code for architectural patterns (dependency injection, event systems, middleware chains) not described in docs
- Check for .env.example, docker-compose.yml, or similar config files that reveal setup requirements not in docs

Report format:

### Context Scout

**Undocumented Environment Variables:**
- `{VAR_NAME}`: used in `{file_path}` — not found in any documentation
  Purpose (inferred): {what it likely controls}

**Undocumented Naming Conventions:**
- `{pattern}`: found in {N} files (e.g., {file1}, {file2})
  Convention: {description of the pattern}

**Undocumented Commands:**
- `{command}`: defined in `{config_file}` — not listed in CLAUDE.md or README
  Purpose: {what it does}

**Undocumented Architectural Patterns:**
- `{pattern_name}`: visible in {files/directories}
  Description: {how it works}
  Why document: {what an agent would get wrong without this}

**Undocumented Setup Requirements:**
- `{requirement}`: found in {config_file}
  Impact: {what breaks without it}

**Summary:**
- {N} env vars, {N} conventions, {N} commands, {N} patterns, {N} setup items undocumented
```
