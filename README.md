# MTG Online

### Paper-style Magic. A better digital table. Your friends already there.

A browser-first Magic: The Gathering platform built around a simple idea: **the fastest, cleanest way to play with the people you already hang out with.** Import a deck, invite your friends, and settle into a responsive multiplayer table—in a browser or directly inside Discord.

**Status: idea parked · discovery planned · no playable application yet**

[Explore the backlog](https://github.com/markbmullins/MTG-Online-game/issues) · [Planning project](https://github.com/users/markbmullins/projects/2) · [Product vision](docs/VISION.md) · [How to resume](docs/PLANNING.md)

## The moment we want to create

Someone in a Discord voice channel says, “Anyone want Commander?”

They open a table. Three friends join. Their decks are ready. Voice is already running. A clear, comfortable Magic table fills the screen.

That is the product: less setup, fewer menus, more time playing.

## What would make it worth building?

- **A table that feels natural.** Fast card movement, readable boards, useful previews, accessible controls and sensible Commander bookkeeping.
- **Discord as a front door.** Activities, invitations and looking-for-game flows bring play into existing communities. The standalone browser remains a first-class way to join the same games.
- **Paper-style freedom.** Start with manual play: humans resolve the rules, while the app keeps shared state consistent and private information private.
- **Communities with their own tables.** Over time, server leagues, deck sharing, spectating and match history could make a Discord server feel like a local game store.

The design question: **How much friction does this remove from playing Magic?**

## Start small, keep the bigger idea

| Horizon | Intended outcome |
| --- | --- |
| Discovery | Validate the first playtest, Discord constraints and Forge integration feasibility. |
| Private-alpha candidate | Four friends import decks, join a private room, play manual Commander and recover from a dropped connection. Explore Discord launch alongside browser access. |
| Later | Better deck tools, public discovery, community tables, spectating, leagues, replays and personalization. |
| Conditional | Optional rules-enforced games if engine discovery supports them. A custom engine only if evidence justifies the investment. |

These are planning horizons, not release commitments. There are no delivery dates or active implementation assignments.

## Rules: freedom first, evidence before commitment

**Manual play**, **rules assistance** and **rules enforcement** are different experiences. Helpful counters, reminders and shortcuts can improve the table without claiming to know every legal action.

[Forge](https://github.com/Card-Forge/forge) is a candidate for a future authoritative engine. An early [discovery epic](https://github.com/markbmullins/MTG-Online-game/issues/1) will test headless four-player play, external choices, private state projections, reconnect, isolation, difficult interactions, maintenance and licensing questions.

Its deliverable is a small end-to-end prototype and an **adopt / reject / conditional-adopt decision**. React and TypeScript are the chosen application stack. Forge adoption and a custom rules engine are not decided. See the [technical architecture](docs/ARCHITECTURE.md) and [API comparison](docs/decisions/002-application-api.md) for the current recommendations.

## Deliberate boundaries

The initial direction is Commander with friends, not a full competitive platform. We would use Discord's existing social environment rather than build voice chat. Browser and Discord clients should share the game experience and session model. Rules-engine internals should stay behind a product-owned protocol.

This repository currently contains planning documents only. There is nothing to install or run, and no implementation agent is authorized by the backlog itself.

## Pick this up later

Read [the vision](docs/VISION.md) and [the planning guide](docs/PLANNING.md). Then [choose the first playtest boundary](https://github.com/markbmullins/MTG-Online-game/issues/52) or refine one discovery question. Collect evidence before committing to architecture. GitHub Issues holds planned work; the Project makes it browsable.

---

An independent, unofficial project concept; not affiliated with or endorsed by Wizards of the Coast. The repository name is a working name. Product naming, card-data/art usage and software licensing remain open decisions.
