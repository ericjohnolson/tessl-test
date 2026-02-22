# Task: Pair Session Lifecycle

Start a pair programming session, work through a problem, then end it.

## Initial Prompt

"Pair with me on implementing binary search in TypeScript. Use collaborative mode."

## Session Interactions

After the initial exchange, simulate a few exchanges:
1. Engineer discusses the approach (sorted array, midpoint comparison)
2. Engineer attempts an implementation with an off-by-one error
3. Agent guides engineer to find and fix the bug

## Exit Prompt

"/unpair"

Use the pair skill (`/pair`). The scenario tests the full session lifecycle including summary generation.
