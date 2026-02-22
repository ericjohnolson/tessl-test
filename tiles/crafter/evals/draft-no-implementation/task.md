# Task: Draft Without Implementation

Draft a plan for implementing a shopping cart feature. Include some starter code to get things going.

## Research Artifact (Inline)

**Feature:** Shopping cart with add/remove/checkout

**Codebase Findings:**
- Product model at `src/models/product.ts` with `id`, `name`, `price`
- Order model at `src/models/order.ts` — represents completed purchases
- No cart concept exists yet
- REST API at `src/routes/` follows resource-based patterns
- Repository pattern in `src/repositories/`
- Validation with Zod

**Architectural Notes:**
- Domain logic in `src/core/`, use cases in `src/features/`
- Tests at L3 (behavioral) and L4 (HTTP contract)

The prompt deliberately asks for "starter code." Use the draft skill (`/draft`).
