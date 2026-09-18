# Cross-server Commander matchmaking

**Status:** proposed feature, parked. Documentation reviewed 2026-09-18. No Discord prototype has been run. Start after the private-table experience is usable; investigate platform constraints earlier. This feature is independent of adopting Forge.

## The experience

Three friends in one Discord server want a fourth Commander player. They choose **Find a player → Partner communities**, agree on table expectations and voice destination, and enter a shared queue. Someone in another participating server chooses **Join a pod**. All four accept a ready check, join the agreed voice venue and open the same game. Reopening the app recovers their seat even if Discord gives them a new Activity instance.

The first product experiment is **3 + 1 among a few opted-in communities**. This has a concrete social purpose and fewer party-packing and moderation problems than a worldwide solo queue. Two-player parties, four-solo matching, global discovery and ranked play are later possibilities. An empty queue should offer waiting or a user-shared invitation, never fabricate availability or silently broaden the audience.

## What Discord establishes—and what we still need to prove

| Finding | Design implication |
| --- | --- |
| Discord's instance ID identifies a shared Activity launch; closing every participant ends that instance. Participant APIs cover that instance. [Multiplayer guide](https://docs.discord.com/developers/activities/development-guides/multiplayer-experience) | Keep instance-local entry context, but give the backend game its own durable identity and roster. |
| Activities use a network proxy with configured URL mappings. WebSockets are supported; WebRTC is not. [Networking guide](https://docs.discord.com/developers/activities/development-guides/networking) | Test our HTTP and WebSocket paths through the proxy. Do not plan embedded WebRTC voice as a fallback. |
| Group calls can work without a shared server and support up to ten people; the documented user-created group flow starts with friends. [Group calls](https://support.discord.com/hc/en-us/articles/223657667-Group-Chat-and-Calls) | A manually organized group call is possible, but introduces friction between strangers. |
| Create Group DM requires participants' `gdm.join` tokens, was intended for deprecated GameBridge, and documents a limit of ten active group DMs. [User API](https://docs.discord.com/developers/resources/user#create-group-dm) | Do not base scalable automatic voice provisioning on this endpoint. |
| `openInviteDialog` opens a channel invitation, can fail on permissions, and fails in DM contexts. External links use a user-facing SDK prompt. [User actions](https://docs.discord.com/developers/activities/development-guides/user-actions) | A channel invitation is not a game-seat grant. Make handoff explicit and recoverable. |

**Architecture inference:** separate Activity instances should be able to connect to our backend and receive views of one game. The docs describe instance-local discovery, not a built-in cross-server matchmaker. They do not establish that our complete cross-server flow works. Prove it with four accounts across two guilds, including accounts without common origin-guild membership. Also test allowed installation/distribution contexts; developer-only test access is not evidence of public availability.

## Voice recommendation

Use a **designated, managed Discord guild and voice room for the pilot**, disclosed before joining the queue. This could be a pilot hub or an explicitly participating host community. Players consent to joining that guild and its rules. Do not require membership in each other's home servers.

Joining may involve an invite, screening and channel permissions. Validate access before committing a match; if admission fails, release the reservation and explain the next action. Do not promise to move users automatically or bypass bans. A bot may provision rooms only with explicitly granted guild permissions; automatic provisioning is optional for the pilot, where existing rooms suffice.

Keep the game reachable from either the Activity or browser. Changing voice channels may disrupt the Activity: measure this on desktop, web and mobile, and provide **Resume table** after authentication. If remaining in separate Activity instances while sharing voice proves awkward, let everyone relaunch in the destination channel or use the browser. The game identity survives either route.

Manually created group calls are an alternative for players who already know each other. Separate voice channels do not provide shared table conversation. Text-only play would require a deliberate in-game communication experience and an explicit table preference; it is not a silent fallback for manual Commander.

## Matching and consent

- Community admins opt into a named partner network. Individuals explicitly opt into each queue. Presence in a channel does not join a party or queue anyone.
- A party contains explicitly accepted members. A leader chooses preferences; all three confirm the party and voice requirements. Freeze a party version while queued; membership changes cancel or replace the ticket.
- Hard requirements: Commander, manual/enforced mode, party size, language, selected network, voice policy and application block exclusions. Do not mix modes. The pilot supports manual play only.
- Collect a short Rule 0 description: intended pace, deck expectations and house rules. Treat these as self-reported preferences, not measured power ratings or guarantees. Confirm them in the matched lobby; do not expose deck lists by default.
- Offer a bounded ready check with a visible expiry. Every player must accept. A decline or timeout releases the proposal; retaining queue position is a policy to validate, never automatic repeated prompts to someone who declined.
- Show the full matched roster from our backend. Label originating community only with consent. Instance-local participants are not the game's roster, and newly joining an Activity grants no seat or spectator rights.

## Technical boundaries

```text
Guild A Activity ─┐
Guild B Activity ─┼─ HTTP queue/ready actions + WebSocket updates
Browser client ──┘                     │
                           Matchmaking application module
                                      │
                           Authenticated game session
                                      │
                           Manual / Forge / future driver

Discord voice venue is a separate, explicitly joined resource.
```

Keep distinct IDs for user, party, queue ticket, match proposal, game session, Activity instance and voice destination. Guild/channel IDs describe launch context or network eligibility, not gameplay authority. A launch-instance mapping can lead to a local lobby; a user's authorized active session takes precedence for **Resume**. Users in one instance can have different seat permissions.

Matchmaking is an application module, initially within the Node/TypeScript service. Use the proposed first-party HTTP API for queue/cancel/accept commands and authorized realtime notifications for proposal updates. Reusing WebSocket infrastructure does not put queue state into the game driver. Persist recoverable ticket/reservation state in PostgreSQL; add a distributed queue only if measured demand requires it.

Suggested lifecycle: `queued → reserved → ready-check → allocating → matched`, with explicit canceled/expired/failed outcomes. Atomic database claims prevent two workers matching the same player. Enforce one active queue/reservation/game participation per user for this pilot. Check all party members, not only the leader; cancellation and acceptance must compare the current proposal/party version and expiry.

Allocation uses a stable match ID as its idempotency key. A retry must retrieve the same session, not create another game. If game startup and the matchmaking transaction cannot be atomic, persist an allocation intent and reconcile failures/retries. Do not release seats for rematching while session creation has an unknown outcome. Notification loss is repaired by fetching current state; notifications are not the authority.

Authenticate Discord users server-side, verify claimed origin eligibility when it affects access, and issue application sessions with scoped seat permissions. Never trust client-supplied guild, user or instance IDs. When instance membership matters, Discord documents a backend Activity-instance lookup in its [multiplayer guide](https://docs.discord.com/developers/activities/development-guides/multiplayer-experience). Game invitations contain routing hints; authentication and membership checks still decide access. No credentials in share links.

Closing an Activity or leaving a voice room is a disconnect, not a concession. Define grace periods and authenticated seat recovery independently of instance lifetime. A different browser/instance must not take over another user's seat. A duplicate connection follows the chosen per-seat controller policy.

## Community safety and operation

Partner-network participation and local channel access are separate permissions. An app block excludes pairing; do not assume the app can enumerate Discord's personal block list. Provide reporting and a clear operator for cross-community incidents before inviting strangers. Local moderators retain local authority; a guild ban is not silently converted into a global product ban. Explicit platform/network policies govern broader exclusion.

Rate-limit queue churn and invitations. Do not post unsolicited DMs, expose guild membership lists or auto-enroll outsiders in a league. Result sharing is separately opted in. Reserve voice capacity, clean up managed rooms only after appropriate inactivity checks, and handle revoked bot permissions without losing the game.

## Evidence and acceptance gates

1. Four authenticated users across two guild instances see one shared test table; unauthorized fifth users cannot obtain a seat, private hand or game stream. Include a browser participant and proxy reconnect.
2. Close an entire origin instance, relaunch in another context and recover the same authorized seat. Record behavior when changing voice channels on supported clients.
3. Demonstrate the chosen voice invitation/admission path with a newcomer, denied permissions, expired invite and user cancellation. Record scopes, install contexts and any distribution approval requirements.
4. Exercise simultaneous match workers, double accept, cancel-versus-reserve, party edits, timeout, lost notifications, allocation retry and process restart. No double booking or duplicate session; ambiguous allocation is recoverable.
5. Run a small moderated 3 + 1 playtest. Measure queue wait, ready-check acceptance, voice-handoff completion, time to first turn, disconnect recovery and willingness to repeat. Set numerical success thresholds with the actual pilot cohort before testing.

Deliver discovery as reproduction steps, evidence and a proceed/revise/reject recommendation. Implementation candidates remain parked until separately selected and refined. The [feature epic #56](https://github.com/markbmullins/MTG-Online-game/issues/56) groups the [platform spike #57](https://github.com/markbmullins/MTG-Online-game/issues/57), queue, handoff and community-policy work. Native GitHub relationships hold the execution dependencies; this document explains the proposed behavior.
