# Task: Research Depth Level Inference

Research how authentication is implemented in this codebase.

## Fixture Codebase Context

The project has the following auth-related files:

- `src/auth/authenticator.ts` — Main authentication service using JWT
- `src/auth/middleware.ts` — Express middleware for route protection
- `src/auth/types.ts` — Auth-related TypeScript interfaces
- `src/routes/login.ts` — Login endpoint
- `src/routes/register.ts` — Registration endpoint
- `src/config/auth.ts` — Auth configuration (token expiry, issuer)

The project uses Express.js with TypeScript, JWT for tokens, and bcrypt for password hashing. Authentication touches a single, well-bounded module.

Use the research skill (`/research`) to investigate.
