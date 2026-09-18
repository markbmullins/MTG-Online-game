# ADR-001: Use React and TypeScript for the application

**Status:** Accepted direction — owner instruction, 2026-09-18. No implementation has begun.

## Context and decision

The owner explicitly chooses React and TypeScript despite earlier comparisons with other frameworks and languages. Use React for the shared browser/Discord game experience and TypeScript for first-party application code and shared client-facing contracts.

Recommend Node.js for application services and a TypeScript manual game driver. Specific build, routing, state-store, database-access and server libraries remain recommendations until the first vertical slice is scoped.

## Consequences

Do not reopen a React-versus-Solid or TypeScript-versus-Go evaluation without a new concrete constraint. Performance concerns should produce measurements and targeted changes.

This choice does not require a TypeScript comprehensive rules engine. A Java Forge adapter is compatible with the application stack. Keep engine transport language-neutral, private state server-side and React independent of the selected engine.

A custom TypeScript rules engine remains conditional. This decision selects the application stack; it does not endorse a third-party port or validate its correctness, licensing or performance.
