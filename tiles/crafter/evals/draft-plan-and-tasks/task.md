# Task: Draft Plan and Task Graph

Draft an implementation plan for adding a user registration feature to this Express.js project.

## Research Artifact (Inline)

**Feature:** User registration with email verification

**Codebase Findings:**
- Existing auth module at `src/auth/` uses JWT tokens
- User model at `src/models/user.ts` has `id`, `email`, `passwordHash`, `createdAt`
- No email verification exists — new capability needed
- Database uses PostgreSQL via Knex; migrations in `src/db/migrations/`
- Existing routes follow pattern: `src/routes/{resource}.ts`
- Repository pattern established in `src/repositories/`
- Validation uses Zod schemas in `src/validation/`

**Architectural Notes:**
- Hexagonal architecture: domain → application → adapters
- L3 tests use Testcontainers for real DB
- L4 tests use Supertest for HTTP contracts
- No mocking of internal dependencies

**Open Questions:**
- Email service: use Nodemailer with SMTP or a third-party API?
- Token storage: in DB table or Redis?

Use the draft skill (`/draft`) to create the implementation plan.
