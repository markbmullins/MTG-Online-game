# Agent guidance

This repository is an intentionally parked product idea. Read README.md, docs/VISION.md, docs/ARCHITECTURE.md and docs/PLANNING.md before proposing or implementing work.

- GitHub Issues is the work authority. Use native parents and blocked-by relationships; Project fields own Theme and Horizon.
- Every seeded issue is a product candidate, not an executable assignment. Work only on the issue or scope explicitly requested by the user; do not select the whole backlog autonomously.
- Epics organize outcomes and are not single implementation tasks. Refine or split candidates before coding.
- Manual play is the initial product direction. React and TypeScript are accepted application choices. Forge adoption, specific libraries, identity model, licensing and release scope remain undecided.
- Preserve hidden-information boundaries in projections, protocol payloads, logs, storage and replays. Manual rules do not mean client-authoritative state.
- Spikes should produce reproducible evidence and a decision, not expand into production systems.
- No application, build commands, test suite or deployment exists yet. Do not invent successful verification or setup instructions.
- ADR-002 recommends tRPC for the first-party HTTP API but remains proposed. Keep engine integration independent of the application API and avoid imposing a shared TS reducer on Forge.
- Record accepted architectural decisions in docs/decisions/ when there is evidence to decide.
