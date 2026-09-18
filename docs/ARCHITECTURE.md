# Technical architecture direction

**Status:** reviewed design direction, not an implemented system. React and TypeScript are accepted choices in [ADR-001](decisions/001-react-typescript.md). Other recommendations remain subject to focused discovery. The product is still parked.

## Start with these boundaries

Build a modular application with a small deployment footprint. Separate responsibilities in code before separating them into services. The application must work with manual play and no comprehensive rules engine; it should also accommodate a Forge adapter or a future TypeScript engine.

| Boundary | Owns | Must not own |
| --- | --- | --- |
| React table and browser/Discord shells | Rendering, accessible interaction, local previews, environment integration | Secret state, rules authority, engine internals |
| Application services | Accounts, decks, room metadata, invitations, history and authorization policy | A competing copy of live gameplay state |
| Game session runtime | Authenticated seats, ordered command admission, engine lifecycle, delivery/reconnect and persistence coordination | Reimplementation of an adopted engine's rules |
| Game driver | Authoritative gameplay state, allowed operations and choice progression for the selected mode | React, public HTTP routing or socket connections |
| Viewer projection | Product-owned, authorized representation for a player or spectator | Blind forwarding of internal engine events |
| Persistence adapter | Durable application records and explicitly supported game recovery artifacts | An assumption that every driver can serialize/restore |

Proposed request path: React → HTTP application API or WebSocket game gateway → application service/session runtime → selected game driver. Responses cross an authorization/projection boundary before reaching clients. HTTP and WebSocket are distinct contracts; they need not require separate deployments.

Recommend Node.js/TypeScript for application services and the manual driver, PostgreSQL for durable data, and a shared React DOM table. A Java Forge worker is added only for a successful integration experiment. Redis, a message broker, Kubernetes, binary transport and a separate gateway service are not starting requirements.

## An engine seam that does not force Forge to become our reducer

Two materially different designs were considered:

| Design | Advantage | Cost |
| --- | --- | --- |
| Every engine implements `applyCommand(state) -> events`, with a shared canonical TypeScript state | Simple manual-engine unit tests and uniform replay assumptions | Forces Forge's opaque state, blocking choices and recovery semantics into a second rules model; likely creates two authorities |
| A stateful game driver with explicit commands, pending choices, projections and capability reporting | Fits a manual engine, in-process TS engine or remote Java session without exposing internals | Requires lifecycle, cancellation, failure and version contracts; restore/replay remain driver-specific |

**Recommend the stateful driver boundary.** A TypeScript implementation can use pure reducers internally. That is an implementation technique, not an obligation imposed on Forge. A normalized client view is a read model, never sufficient state from which to reconstruct an arbitrary engine.

Define the contract using language-neutral, versioned data schemas; generate or validate TypeScript types and Java mappings from that contract. Do not use an inferred tRPC router type as the Java integration protocol. JSON over a simple internal transport is a reasonable first experiment; choose gRPC only if measured integration needs justify it.

The driver contract should cover:

| Operation or result | Required meaning |
| --- | --- |
| Create / end session | Fixed mode, engine build/card-data versions, players, deck snapshots and cleanup |
| Submit intent | Authenticated actor supplied by the runtime, request ID, validated mode-specific command and preconditions |
| Pending choice | Opaque choice ID, entitled responder(s), version, structured options and constraints; no engine class names |
| Answer choice | Validate responder, current choice and answer; distinguish rejection, progression, another choice and terminal failure |
| Observe state | A consistent engine observation mapped to authorized product views, including explicit pending decisions |
| Capability report | Supported commands/choice kinds, formats, correction behavior, snapshot/restore and replay support |
| Checkpoint / restore | Optional, versioned and evidenced; unsupported is a real result, not a stub that pretends to succeed |

The internal engine observation is privileged. With Forge, filtering may begin inside the Java adapter because that is where visibility is known. A generic TypeScript layer cannot safely infer secrecy from field names. Public schemas, adapter conformance fixtures and negative privacy tests constrain every implementation.

Manual commands such as moving a card and enforced commands such as casting a spell share an envelope, not identical semantics. An enforced session may reject a manual move. UI controls follow explicit mode/capabilities. Unsupported choice types should produce a clear unsupported state, not quietly select an answer. Changing engines or switching manual/enforced mode mid-game is outside the design guarantee.

A waiting choice releases execution resources where the engine permits it. The session remains responsive to authenticated choice answers, reconnect and cancellation while unrelated gameplay inputs are rejected or held under a defined policy. Do not hold a mailbox handler awaiting a player response that can only arrive through that same blocked mailbox. Forge may require a dedicated execution thread; test this, including cancellation and leaked-thread cleanup.

## Three kinds of state on the client

1. **Application data:** decks, profile and room listings in an HTTP query cache.
2. **Authorized live-game view:** normalized objects/zones and pending choices, updated atomically from the game protocol.
3. **Transient interaction:** drag position, selection, hover, camera and animations; local to the client unless explicitly shared as disposable presence.

Use granular subscriptions and stable snapshots for the live view. React's external-store API requires cached/immutable snapshots; a changing snapshot object on every read is incorrect. Zustand with selectors is a reasonable candidate, not a mandatory dependency. Avoid duplicating gameplay state in both the query cache and table store. [React external-store guidance](https://react.dev/reference/react/useSyncExternalStore).

Start with DOM cards, CSS transforms, accessible controls and explicit image loading/decoding policy. Profile render/commit, layout, paint, image memory and garbage collection rather than assuming React itself is the bottleneck. Thousands of composited layers can also be expensive. A renderer boundary preserves the option of a canvas surface if measurements demand it; accessibility and interaction semantics still need an implementation.

Animate a drag locally and submit one meaningful final command. Optional drag previews are rate-limited, bounded, disposable and visibility-filtered. Do not broadcast private hand selections or hover activity by default. Cross-zone movement, tapping and counters still require server acceptance; optimism must be reversible and permitted by mode. Never predict secret card identities or random results.

## HTTP API choice

**Recommend tRPC over HTTP for the first-party TypeScript application, ordinary HTTP endpoints for external integrations, and a separate explicit WebSocket game protocol.** [ADR-002](decisions/002-application-api.md) compares REST/OpenAPI, tRPC and GraphQL and keeps this recommendation proposed until implementation scope is confirmed.

Business logic belongs in application services callable from any transport. Discord handlers call those services locally or through the application API when deployed separately. OAuth callbacks, provider interactions and future public integrations retain their required HTTP semantics. Forge communicates through the engine contract, independent of the application API choice.

Creating a room is application work. Starting a game crosses into session ownership: validate readiness and immutable deck snapshots, use an idempotent start request and record the resulting session identity. Once a session exists, HTTP endpoints must not mutate live state behind the session queue. Result publication should use a durable idempotent handoff/outbox if it crosses a persistence or service boundary.

## Discord context is not game identity

The [cross-server matchmaking proposal](features/CROSS_SERVER_MATCHMAKING.md) keeps guild, channel, Activity instance, party, match proposal, game session and voice destination distinct. Matchmaking is application work; authenticated seats and viewer authorization belong to the session runtime. Multiple launch contexts may lead to one game, and a new Activity instance must not erase an existing seat. Cross-instance play and voice handoff require a focused platform spike before adoption.

## Ordering, privacy and delivery

A command is a request. An internal transition is what the driver committed. A viewer update is the authorized representation of that transition. Presence is disposable. None of these is interchangeable with a rules engine's internal “event” during replacement/trigger processing.

Use one authoritative execution owner per game and an actual serial mailbox. `async handle()` does not serialize concurrent calls across `await`. Process different games independently; avoid a global queue. When owners can fail over, persist an ownership epoch/fencing token and reject stale-owner writes and responses. A hash modulo current worker count is not a safe migration protocol.

An illustrative client envelope includes protocol version, session ID, command ID and command-specific preconditions. The actor comes from verified connection/session identity, not a trusted payload field. Define:

- **Deduplication:** scope by session and actor; keep payload identity and previous outcome. Reuse with different content is an error. Preserve deduplication through the advertised retry/recovery window; do not promise universal exactly-once delivery.
- **Preconditions:** choice/object versions for semantic validity. A blanket global revision check rejects independent legitimate actions too often. Safe deltas can be revalidated against current state; stale choices and invalid object references must fail.
- **Delivery:** sequence each authorized stream, attach snapshot/stream epochs and define atomic update batches. On a gap, resume from retained authorized updates or replace the view with a fresh snapshot. Avoid racing subscription with snapshot creation.
- **Privacy:** do not broadcast canonical state/events, private choices, random seeds or raw error details. Prefer counts for unknown zones. When handles are needed, make them viewer-scoped and invalidate their linkage when knowledge is lost. Stable hidden IDs can expose tracking through shuffles and concealment.
- **Projection lifecycle:** reset cached views on seat/role changes, removals and session changes. Historical replay permissions may differ from live permissions. Explicitly document residual timing/traffic side channels; never expose global internal sequence gaps as a convenient public cursor.
- **Backpressure:** bound message size, command queues, outgoing buffers and pending requests. Coalesce/drop presence first; never silently drop committed gameplay updates—resync or disconnect slow consumers. The browser WebSocket API does not supply automatic backpressure. [MDN WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket).

Validate runtime schemas, authorization and input sizes at trust boundaries, including the engine adapter. TypeScript alone does not validate network or stored/versioned artifacts. WebSocket authentication must address origin checks, credential expiry and revocation, not just the first connection. Keep tickets short-lived and scoped if used; avoid long-lived credentials in URLs/logs.

## Persistence, replay and randomness are separate contracts

Choose recovery guarantees before storage shape. For a manual driver, a practical candidate is a durable transition journal plus versioned checkpoints in PostgreSQL. This is a proposed game-session mechanism, not a mandate to event-source the whole application. Disk durability depends on actual database commit/configuration, not on calling an API named `persist`. [PostgreSQL WAL](https://www.postgresql.org/docs/current/wal-intro.html).

For a driver that supports staging, compute a transition, atomically commit its durable record/deduplication outcome and revision, then acknowledge/broadcast. On failure, discard staged work. If an opaque Forge operation has already mutated in-memory state, a failed commit cannot simply be retried or rolled back: suspend the session, resolve commit ambiguity and restore from a proven artifact or declare the recovery limit. An intent journal alone does not prove exactly-once engine execution.

Document crash cases: before mutation, after mutation/before commit, commit succeeded but acknowledgement lost, and checkpoint/owner failure during a pending choice. If the only proven Forge guarantee is reconnect to a resident process, state that clearly. A projection log can support visual playback while remaining insufficient to resume the engine.

Distinguish:

- **UI playback:** historical authorized views or sufficient presentation events.
- **Engine replay:** reconstructing the same engine state under pinned engine build, card data, configuration, choices, randomness and external/time inputs.
- **Crash recovery:** restoring an authoritative continuation with ownership and deduplication intact.
- **Undo:** a product policy; information already revealed cannot be unlearned. Enforced-mode takebacks require engine support.

Use unpredictable server-side randomness for live games. Deterministic test seeds are valuable; they must not become publicly recoverable live shuffle seeds. A seeded non-cryptographic generator is not automatically safe. Persist protected random outcomes/state as required by the chosen replay strategy; a seed alone is insufficient across changing engines/data or unrecorded nondeterministic inputs.

Redis can later help discovery/presence/routing. It is not necessary for a first process, and Pub/Sub has at-most-once delivery, so it cannot be the sole recovery journal. [Redis delivery semantics](https://redis.io/docs/latest/develop/pubsub/).

## Performance: measure workloads, isolate expensive work

React/TypeScript is a productive starting choice, not a capacity claim. A turn-based game can generate expensive bursts, large projections and slow clients. Synchronous work blocks other sessions on the same Node event loop; wrapping it in `async` does not fix that. Use process or worker-pool isolation for measured CPU-heavy TS work and process isolation for a Java engine as appropriate. One game's rules transitions remain ordered. [Node event-loop guidance](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop), [worker threads](https://nodejs.org/api/worker_threads.html).

Benchmark ordinary tables, dense Commander boards, mass tokens/triggers, expensive choices, spectators, concurrent games, reconnect storms and slow consumers. Record hardware, engine/data revision and workload. Measure p50/p95/p99 command-to-visible latency separately from engine execution, queue wait, database commit and projection/serialization; include memory per resident game, GC pauses, event-loop lag, client frame time, image memory and payload sizes.

Set budgets from the first playtest environment; the supplied 10/50/250 ms and “500 games per process” numbers are not measured requirements. Optimize allocations, copying and derived-state caching only against profiles and correctness tests. Worker time/memory limits contain pathological processing; they must suspend/report failure rather than silently skip required game rules.

## Proposed code organization

Begin with a few cohesive modules: application services, session runtime, protocol/projection schemas, manual driver, shared game UI and platform adapters. They may live in one TypeScript workspace. Add the Forge adapter/service only when its experiment needs it.

Domain logic should not depend on React, tRPC, database clients or socket libraries. Transport adapters map into domain requests and out to wire views; the wire format does not become the engine's internal object model. Browser imports must exclude server-only data and code. Avoid a miscellaneous `shared` package that gradually exposes everything everywhere.

## Next decisions and evidence

GitHub issues remain the work authority. The next candidates are the [HTTP API decision](https://github.com/markbmullins/MTG-Online-game/issues/53), [game-driver contract](https://github.com/markbmullins/MTG-Online-game/issues/54) and [performance baseline](https://github.com/markbmullins/MTG-Online-game/issues/55). Existing protocol, recovery, UI and Forge tickets include the specific failure cases above. Preserve custom-engine semantics as [conditional research](TECHNICAL_REVIEW.md), not a prerequisite for manual Commander. No implementation is authorized by this document.
