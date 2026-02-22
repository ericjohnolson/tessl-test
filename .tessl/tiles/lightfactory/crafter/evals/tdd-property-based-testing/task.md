# Task: TDD with Property-Based Testing

Using TDD with property-based tests, implement a `Money` module with:

- `add(a, b)` — adds two integer amounts
- `subtract(a, b)` — subtracts b from a
- `split(amount, parts)` — divides an integer amount into N parts where:
  - The sum of all parts equals the original amount exactly
  - Each part is a non-negative integer
  - Parts differ by at most 1 (fair distribution)

Use `fast-check` for property-based testing. Properties should be preferred over example-only tests for core logic. Use the TDD skill.
