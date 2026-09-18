# Product vision

## Who this is for

Start with friends who already play Commander together, especially groups that organize in Discord. The hypothesis is that a better table and a shorter path from conversation to game matter more initially than comprehensive rules automation.

The experience should retain the social flexibility of paper play: shortcuts, takebacks, Rule 0 agreements and conversation. Manual play still needs server-owned state, authenticated seats and strict private-information boundaries.

## The community loop

A community installs the app → someone opens an LFG invitation → friends join a table → the shared client opens inside Discord → players use existing voice → an opted-in result or league update invites the next game.

This is a distribution hypothesis to test, not an established acquisition channel. Avoid promises about setup time, conversion or supported Discord environments until measured.

The browser is an independent entry point into the same sessions. Discord should supply its existing voice, chat, roles and community context. Product-specific game permissions and safety remain the application's responsibility.

## Capability map

These are enduring themes, not never-ending epics. GitHub tracks bounded outcomes within them.

| Theme | Ideas to preserve |
| --- | --- |
| Core Play Experience | Zones, card movement, turn controls, counters/tokens, Commander, corrections/history, spectators and game recovery. |
| Decks & MTG Data | Card data, import/export, builder, printings/art, legality, folders/tags, sharing, versions and playtesting. |
| Multiplayer Platform | Rooms, invites, settings, public/private discovery, LFG, presence, friends/recent players and persistent sessions. |
| Discord Integration | Account linking, Activity, app commands, join/watch flows, configured channels, persistent tables, deck embeds and opt-in role integration. |
| UX & Table Quality | Drag/drop, smart placement/grouping, zoom/pan, previews, context actions, arrows, visual stack, readable log, shortcuts, accessibility and touch layouts. |
| Social & Community | Profiles, groups, events, reputation, moderation, admins and opt-in public community pages. |
| Organized Play | Leagues, weekly pods, standings, Swiss/brackets, timers, results disputes, judges, deck registration, draft and cube nights. |
| History & Replays | Recorded events, replay timeline/sharing, match history and player/deck/matchup statistics. |
| Platform & Reliability | Identity, permissions, protocol, session ownership, storage, crash recovery, scaling, diagnostics, abuse controls, deployment and backups. |
| Customization | Playmats, backs/sleeves, themes, avatars, sound, layouts, hotkeys and preferred actions; possible cosmetic monetization. |
| Rules Assistance | Optional turn/untap helpers, mana, tokens, keywords, Commander bookkeeping, monarch/initiative/city blessing, day/night and reminders. |
| Rules Engine | Forge discovery, possible enforced games, conformance tests and a custom engine only if justified. |

## A first playable hypothesis

Four friends can import decks, create and join a private room, draw and move cards, manage Commander bookkeeping, communicate actions, correct mistakes and reconnect after a network drop. They can understand the table without coaching. Discord launch is an early distribution experiment.

Before implementation, choose supported environments, identity behavior and the exact playtest boundary. Record baseline setup time, common-action effort, confusion points, synchronization failures and willingness to play again. Set success thresholds when an actual test group and prototype exist.

Public matchmaking, tournaments, elaborate deck building, monetization and complete rules automation are outside that first hypothesis.

## Architecture direction

One shared game UI can live in browser and Discord shells, with small platform adapters for identity, invitations and environment context. A server owns sessions, commands and viewer-specific state. An engine adapter keeps Forge-specific classes out of the public protocol.

React and TypeScript are the owner-selected application stack; see [ADR-001](decisions/001-react-typescript.md). The [technical architecture](ARCHITECTURE.md) recommends Node.js application services, a stateful engine-driver boundary and an explicit WebSocket game protocol. [ADR-002](decisions/002-application-api.md) proposes tRPC for the first-party HTTP API. Specific libraries, deployment topology and Forge adoption remain open. A Java engine is compatible with the chosen application stack.

Manual mode should not require Forge. Optional assistance must state its limits. An enforced mode may have different correction, unsupported-card and house-rule behavior. Do not assume that a rules engine can safely resume after arbitrary manual edits.

## Forge discovery

Prove: create a four-human game without the GUI → cast a spell → serialize a target choice → answer in a browser → opponents pass priority → resolve → update four authorized views.

Investigate every class of player choice, command zones and damage, isolation/static state, resource costs, session residency versus restore, a 20–50 scenario rules corpus and upgrade maintenance. A successful demo alone is insufficient evidence of general engine suitability.

The final decision must show passed and failed gates, limitations and estimated integration/maintenance cost. Rejection is a useful discovery outcome. Conditional adoption must identify unmet gates explicitly.

Research component licenses and actual distribution models before an adoption/commercialization decision. Service separation is an architectural option, not a legal conclusion. Card data, artwork and product naming are separate questions. No repository license or ownership strategy is selected here.

## Where these ideas came from

This plan synthesizes the owner's supplied conversations on Discord distribution, twelve capability areas, Forge integration and a portable GitHub Issues model, plus the explicit Forge discovery epic. Broad brainstorms are retained as candidates; speculative claims about coverage, schedules and provider capabilities are not treated as verified facts.

Useful primary starting points for future discovery:

- [Forge repository](https://github.com/Card-Forge/forge) and [card scripting documentation](https://github.com/Card-Forge/forge/wiki/Card-scripting-API).
- [Discord developer documentation](https://discord.com/developers/docs/intro).

Recheck upstream code, terms and platform documentation when the project resumes; this document records intent rather than a completed technical or legal assessment.
