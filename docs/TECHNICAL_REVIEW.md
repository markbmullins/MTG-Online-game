# Review of the early technical discussions

Reviewed 2026-09-18. This is a curated record of useful reasoning and corrections, not an implementation specification or an endorsement of all claims in the source conversations. The resulting direction is in [ARCHITECTURE.md](ARCHITECTURE.md).

## Keep, change, defer

| Original idea | Assessment |
| --- | --- |
| React with granular game-state subscriptions; DOM cards first | Keep. Separate authoritative views, HTTP cache and ephemeral interaction. Profile images/layout as well as renders. |
| HTTP app API plus WebSocket gameplay | Keep the responsibility split. Prefer tRPC for the first-party TS surface; preserve ordinary HTTP integration routes. |
| Commands, events, presence | Keep, but distinguish internal engine operations, durable records and filtered client updates. Raw engine events must not be broadcast. |
| Framework-independent reducer as the universal engine interface | Keep reducers as an option inside the manual/TS driver. Replace the universal constraint with a stateful engine boundary. |
| One actor/owner per game | Keep. Add a real serial queue, bounded work and ownership fencing when failover exists. The sample `async handle` alone is not an actor. |
| Redis stores live state from day one | Defer. Resident ownership plus explicit durable recovery can start without Redis. A cache or Pub/Sub channel does not supply recovery guarantees. |
| Snapshot + event log yields replays “for free” | Correct. Playback, privacy, compatibility, engine restoration and undo are separate problems. |
| Global revision mismatch for every command | Refine. Separate view delivery cursors from semantic preconditions; validate commands against current authority. |
| TS is definitely fast enough; hundreds of games per process | Treat as unmeasured hypotheses. Use workload benchmarks, tail latency and event-loop/resource measurements. |
| Stable hidden-card IDs | Unsafe as a blanket rule. Avoid tracking secrets through randomized or concealed zones. |
| Use engine/card support counts as evidence of readiness | Reject as sufficient evidence. Audit actual scenario coverage, choice semantics, restore and resource isolation. |
| Build a complete custom TS rules engine first | Superseded by manual-first product direction and early Forge discovery. Preserve only a conditional research outline. |

## Conditional custom-engine research

If a later decision justifies a custom engine, keep the ideas of composable card semantics, explicit choices, injected randomness/time, object identity distinct from printed data, and derived characteristics. An extensible DSL and a bounded trusted-code escape hatch can be useful; arbitrary user-supplied executable card scripts are a different security problem.

Do not create a DSL or duplicate layers, replacement effects and card implementations merely to wrap Forge. Let the adopted engine own those semantics. Likewise, a visual stack and manual counters do not imply that the client understands full rules.

A possible custom-engine learning sequence goes from simple two-player scenarios to targeting, priority, combat, triggers, replacements and continuous effects, then complex costs and multiplayer. That sequence applies to engine research only; it does not displace four-player Commander as the product's first audience.

Separate printed/card-data identity, physical card instances, rules-object identity and viewer handles. Tokens, copies, multi-face cards, merged objects and zone-change exceptions mean these are not interchangeable UUIDs. Keep rules identity inside the driver; the view represents only the associations a viewer may know.

## Concrete corrections to the illustrative rules code

The [Comprehensive Rules linked by Wizards](https://media.wizards.com/2026/downloads/MagicCompRules%2020260819.txt) were checked during review. Pin the rules revision again when building fixtures.

- **Giant Growth timing:** a normal 2/2 that has already taken 3 damage dies to state-based actions before a player can cast a rescue spell. Test Giant Growth resolving before Bolt, plus the failure case after damage (117.5, 704.5g).
- **Layer enum:** the sample separate “P/T counters” sublayer is outdated. Modifiers and counters are in 7c; switching is 7d (613.4).
- **Triggers:** detecting a trigger does not generally mean immediately putting it on the stack. Respect the priority checkpoint and APNAP ordering (603.3, 117.5).
- **Targets:** a `creatureOrPlayer` filter is insufficient for “any target,” which also includes planeswalkers and battles (115.4).
- **Mana abilities:** the sample Elvish Mystic ability requires mana-ability semantics, rather than an ordinary stack activation (605.3b).
- **Zone changes:** “new object” is the usual rule, with explicit exceptions; incrementing an ID does not implement all of them (400.7).

Replacement/prevention ordering, simultaneous events, last-known information and loops require dedicated semantics. A convenient event-bus ordering is not proof of conformance. Build tests from current rules and rulings, not from conversational pseudocode. Differential agreement with another engine is evidence, not an independent rules oracle.

## The TypeScript Forge port mentioned in the notes

[Baldugar/mtg-forge-ts](https://github.com/Baldugar/mtg-forge-ts) exists and its README makes broad parity, headless-use and recovery claims. Those claims were observed, not reproduced in this review. Do not record them as verified card correctness or production readiness.

If it remains relevant at engine selection time, pin a revision and inspect what parity actually measures, run independent edge cases, exercise four-human choices and process-failure recovery, inspect maintenance/provenance, and evaluate licensing through the existing assessment. It can be compared as an optional driver candidate without changing the React client. This review neither adopts it nor creates a dependency on it.

## What was deliberately not carried forward

Framework rankings, broad language superiority claims, precise unmeasured throughput/latency estimates, suggested percentages of declarative versus custom card code, and “replays for free” promises do not belong in the durable plan. Preserve the questions and experiments that can establish those facts instead.
