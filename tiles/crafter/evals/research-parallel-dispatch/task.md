# Task: Research Parallel Agent Dispatch

Research how to add a payment processing module to this project. Use the research skill (`/research`).

## Fixture Codebase Context

The project has:

- `src/core/order.ts` — Order domain model with total calculation
- `src/core/discount.ts` — Discount code validation and application
- `src/features/create-order.ts` — CreateOrder use case
- `src/repositories/orderRepository.ts` — Order persistence
- `src/routes/orders.ts` — POST /orders HTTP endpoint
- `src/db/connection.ts` — PostgreSQL connection pool
- `package.json` — Uses Express, Knex, fast-check for testing

The feature requires integrating with Stripe for payment processing, adding a payments table, and creating a PaymentService in the core layer.

Use the `/research` skill to investigate this. The scope spans multiple modules and involves a new external API (Stripe), so standard or deep depth is appropriate.
