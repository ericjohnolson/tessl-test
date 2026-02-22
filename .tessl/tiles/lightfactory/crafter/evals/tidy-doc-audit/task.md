# Task: Tidy Documentation Audit

Run tidy on this project. Use the tidy skill (`/tidy`).

## Fixture Project Context

The project has the following documentation issues:

**CLAUDE.md** contains:
- A reference to `src/utils/helpers.ts` which was deleted 3 months ago
- A build command `npm run build:prod` that no longer exists (renamed to `npm run build`)
- No mention of the `DATABASE_URL` environment variable used in `src/db/connection.ts`

**README.md** contains:
- A link to `docs/architecture.md` which exists and is current (no issue)
- A link to `docs/api-reference.md` which was deleted

**Missing documentation:**
- The project uses a `*Repository` naming convention consistently across 5 files, but this convention is not documented anywhere

**Cosmetic:**
- CLAUDE.md has a "## Testing" section that duplicates information already in the "## Commands" section

The tidy skill should audit, classify findings by severity, and fix approved items with one commit per fix.
