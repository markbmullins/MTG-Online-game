# ADR-002: Prefer tRPC for the first-party application API

**Status:** Proposed recommendation, 2026-09-18. Confirm when the first vertical slice and external API requirements are chosen.

## Recommendation

Use **tRPC over HTTP** for first-party React/TypeScript application queries and mutations. Keep **ordinary HTTP endpoints** for OAuth callbacks, Discord/provider requests, downloads and any deliberately published REST API. Use an **explicit WebSocket protocol** for live game commands, choices and authorized state streams. This is a responsibility split, not a requirement for three deployments.

The browser and Discord Activity share one client; they do not create a need for different GraphQL query surfaces. A TypeScript bot can share application-service calls or use a typed API client. Forge uses a separate engine boundary, so Java integration does not force the application API to be REST.

## Options

| Option | Strength for this product | Cost or tradeoff | Choose when |
| --- | --- | --- | --- |
| REST + OpenAPI + generated TS client | Explicit language-neutral resource contracts; straightforward tooling and public integrations | Schema/client generation and deliberate endpoint composition; type safety is available but not automatic | A stable public API or independently released non-TS clients are near-term requirements |
| tRPC over HTTP | Fast first-party iteration with inferred TS request/response types; fits shared React/TS code | Compile-time coupling and deploy-version skew still need management; weaker fit as a public cross-language SDK contract | First-party TS clients dominate, as currently planned |
| GraphQL | Client-selected nested data and a discoverable graph useful across independent consumers | Schema/resolver and cache decisions, field authorization, N+1 prevention and demand controls | Concrete multi-client query needs justify that machinery |

These are architecture judgments for this scope, not universal performance rankings. REST can return composed screen data, tRPC can batch, and GraphQL can serve small applications. None of them solves game ordering, privacy or recovery.

tRPC offers inference without API code generation, but input/output validation remains explicit. Its HTTP protocol also means non-TS access is possible; it simply does not provide the same inference benefit. [tRPC introduction](https://trpc.io/docs/), [validators](https://trpc.io/docs/server/validators), [HTTP protocol](https://trpc.io/docs/rpc).

OpenAPI provides a language-neutral HTTP interface description. A REST choice need not sacrifice typed clients. [OpenAPI specification](https://spec.openapis.org/oas/latest.html).

GraphQL is technically capable of subscriptions and real-time use. Keeping gameplay separate is a design preference that makes command acknowledgement, resumption and private streams explicit, not a claim that GraphQL cannot support games. If adopted, query depth/breadth, pagination and cost controls would be part of operating it. [GraphQL security guidance](https://graphql.org/learn/security/).

## Contract and migration discipline

Keep authorization and business rules in application services, independent of routers/resolvers. Share only intended client contracts; never expose database rows or privileged engine objects through inferred return types.

Runtime validation, stable error codes, bounded pagination, per-operation authorization, cancellation and request limits apply regardless of transport. Batching is not a database transaction. For non-idempotent mutations, define replay protection and commit semantics. Old browser tabs must either remain compatible for a declared window or receive an explicit upgrade response; a monorepo does not make all deployed clients update simultaneously.

Use one app-data cache in the frontend; TanStack Query with the tRPC integration is a candidate. The live game store remains separate. Do not add Apollo just because the original discussion mentioned GraphQL.

Avoid implementing every feature in both REST and tRPC. Add thin REST adapters only for actual integration contracts. Revisit this recommendation if public ecosystem access or several independently released clients become core requirements. Engine choice alone is not a reason to switch the application API.

## Evidence before accepting

Exercise deck read/update, private deck denial, room creation and idempotent game start through one small vertical slice. Check stale clients, error serialization and an ordinary provider callback. Confirm who consumes the API, then accept this ADR or record the REST/OpenAPI alternative. A three-framework benchmark project is unnecessary.
