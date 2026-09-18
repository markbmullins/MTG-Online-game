# Planning and resuming the project

[GitHub Issues](https://github.com/markbmullins/MTG-Online-game/issues) is the source of truth for planned work. [MTG Online — Idea & Discovery](https://github.com/users/markbmullins/projects/2) is the canonical Project. This repository captures durable vision and decisions, not a second task database.

## Current posture

The idea is parked. All seeded issues are `product-candidate`, unassigned and unscheduled. They are useful future work, not an approved implementation queue. There are no active automations or release commitments. The Project is private; repository issues and documents follow the public repository's visibility.

## Adaptation of the supplied operating model

| Concern | Canonical location |
| --- | --- |
| Work remains / completion reason | Issue state and close reason |
| Kind | One `kind:*` label: epic, feature, task, spike or decision |
| Product area | Project `Theme` single-select field |
| Planning horizon | Project `Horizon`: Discovery, Alpha candidate, Later or Conditional |
| Epic membership and progress | Native parent/sub-issue relationships and progress |
| Hard prerequisite | Native blocked-by relationship |
| Future release commitment | Milestone, only once actual release scope is agreed |
| Parked idea | `product-candidate` label |
| Future autonomous execution | Explicit approval of a refined issue under an adopted agent contract |

This is a personal repository. The small `kind:*` vocabulary is a deliberate fallback for organization-managed issue types. If the repository moves to an organization with configured native types, migrate the classification and remove the fallback labels instead of maintaining both.

Themes are not epics. Epics have finishable integrated outcomes; children hold detailed acceptance criteria. Later feature candidates may still need splitting. A hard dependency means completion truly requires the other outcome, not just that one idea might come first. The Forge decision depends on its research tickets; manual play does not depend on adopting Forge.

Horizon expresses intent, not activity. The Project's built-in Status is not an execution authority and is left unused. Do not duplicate state with lifecycle labels, invent delivery dates or use a milestone as an epic. Assignees remain empty until someone takes responsibility.

## Browse the backlog

The Project has saved views for **All ideas**, **Epics**, **Discovery**, **Alpha candidates** and **Later & conditional**, with Theme, Horizon and native hierarchy columns. Useful issue searches:

- [Epics](https://github.com/markbmullins/MTG-Online-game/issues?q=is%3Aissue%20is%3Aopen%20label%3Akind%3Aepic)
- [Discovery spikes](https://github.com/markbmullins/MTG-Online-game/issues?q=is%3Aissue%20is%3Aopen%20label%3Akind%3Aspike)
- [Decisions](https://github.com/markbmullins/MTG-Online-game/issues?q=is%3Aissue%20is%3Aopen%20label%3Akind%3Adecision)
- [Parked candidates](https://github.com/markbmullins/MTG-Online-game/issues?q=is%3Aissue%20is%3Aopen%20label%3Aproduct-candidate)

Native parent links provide the epic drill-down. Child order is a suggested reading sequence, not permission to execute. No separate child checklists are maintained.

## Resume with one question

1. Read the README, vision and [technical architecture](ARCHITECTURE.md), including accepted versus proposed ADRs. Reconfirm the first audience and playtest boundary.
2. Choose a small discovery ticket, such as headless Forge initialization or Discord Activity constraints. Check native prerequisites.
3. Recheck current upstream documentation and record assumptions. Split work that cannot fit one focused implementation/review cycle.
4. Refine acceptance criteria, exclusions, verification and the expected evidence. Resolve material decisions for dependent implementation.
5. Explicitly authorize the selected work. Only then remove `product-candidate` from that issue. Removing the label alone is not an agent execution contract.
6. Keep experiments small. Record results in a document or ADR with revision, reproduction steps, limitations and recommendation.

An adoption decision may be negative. Close a completed investigation when evidence answers its question; do not falsely claim its proposed technology succeeded. Close rejected ideas as `not planned` with a reason. A prerequisite canceled as `not planned` does not satisfy its dependents.

Close an epic only after resolving required child scope and verifying its integrated outcome. When a decision changes scope, update affected issues and native dependencies rather than leaving misleading blockers.

## Future agent handoff

There is no Ralph installation or `ralph:ready` queue here. Before unattended execution, adopt a concrete ticket/authorization contract and selector that excludes candidates, epics and unresolved prerequisites. Do not let an agent choose freely from the entire backlog.

A useful future instruction is: “Refine issue #N against the vision, inspect its native dependencies, and propose the smallest verifiable experiment.” Authorization to refine does not authorize building every related idea.

Review the backlog when returning to the project. No recurring maintenance process is necessary while it is parked.

## Platform references

Native relationships follow [GitHub sub-issues](https://docs.github.com/en/rest/issues/sub-issues) and [issue dependencies](https://docs.github.com/en/rest/issues/issue-dependencies). The [REST Project view creation API](https://docs.github.com/en/rest/projects/views) targets organization-owned projects. This personal Project uses GitHub's GraphQL view mutations, verified against the live schema during setup on 2026-09-18 UTC.
