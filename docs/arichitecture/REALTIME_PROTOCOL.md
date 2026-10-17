# Red Earth — Real-Time Protocol

**Path:** `docs/architecture/REALTIME-PROTOCOL.md`  
**Status:** Draft — proposed protocol; validate against the existing repository and selected multiplayer framework  
**Scope:** Client/server messages, payload validation, gameplay updates, errors, and protocol evolution.

## 1. Purpose

Red Earth needs a clear contract between its browser client and authoritative game server.

The protocol defines:

- Which messages clients may send.
- Which messages the server may send.
- What each message contains.
- How messages are validated.
- How the server communicates errors.
- How gameplay state is synchronized.
- How protocol changes remain compatible.

This document defines the intended contract, not an assertion that these messages already exist in the codebase.

## 2. Core Rules

1. The server owns authoritative gameplay state.
2. Every incoming payload is untrusted.
3. Every message has a defined payload schema.
4. The server validates identity, permissions, room membership, and current game state.
5. High-frequency gameplay messages should remain compact.
6. Persistent account data must not be confused with temporary match state.
7. Clients must handle rejected actions and unexpected disconnections.
8. Network messages must not expose secrets or unnecessary private data.

The protocol must fit the selected multiplayer framework. If Colyseus is selected, use its state synchronization and room-message conventions where appropriate rather than rebuilding those capabilities unnecessarily.

## 3. Message Categories

Organize messages into logical categories.

| Category | Responsibility |
|---|---|
| Session | Establish and validate the player's connection |
| Lobby | Join, leave, and synchronize lobby state |
| Presence | Communicate relevant online status |
| Party | Invitations and party membership |
| Matchmaking | Enter queues and receive match assignments |
| Gameplay | Submit input and receive authoritative state |
| Combat | Weapon actions and validated combat outcomes |
| Match | Match lifecycle, objectives, and results |
| Chat | Send and receive authorized text messages |
| Emotes | Request and broadcast approved emotes |
| Safety | Reports and player safety actions |
| Errors | Communicate rejected operations and recoverable failures |

Keep event names and responsibilities consistent. Avoid creating several messages that perform the same operation with subtly different behavior.

## 4. Payload Conventions

Use TypeScript types for developer ergonomics and runtime schemas for validation.

A proposed common envelope for ordinary application messages is:

```ts
interface ProtocolMessage<TPayload> {
  requestId?: string;
  payload: TPayload;
}
```

This is a conceptual contract. Do not force an envelope around framework-managed state updates or high-frequency input messages if doing so creates unnecessary overhead.

### Field rules

- Use consistent naming conventions.
- Use explicit units for time, distance, and angles where ambiguity is possible.
- Reject `NaN`, infinite numbers, invalid identifiers, and out-of-range values.
- Set maximum string lengths and array sizes.
- Use enumerated values for known actions and states.
- Avoid accepting arbitrary object properties.
- Never trust client-supplied account roles or authoritative match outcomes.

The server should generate authoritative timestamps and identifiers where required.

## 5. Connection and Session

### Proposed client messages

**`session:ready`**

Indicates that the client has initialized its network handlers and is ready for the next permitted step.

The server must not interpret this as proof of authentication.

**`session:resume`**

Requests restoration of an existing session or match participation, if supported.

The server must validate the session, player identity, expiry, and eligibility to resume. A client-provided player ID is not sufficient authorization.

### Proposed server messages

- `session:accepted`
- `session:rejected`
- `session:expired`
- `session:resume_result`

These names are proposals. The final protocol should use the conventions of the selected framework.

The server must avoid sending session credentials back to the client in ordinary gameplay messages.

## 6. Lobby and Presence

### Proposed client messages

- `lobby:join`
- `lobby:leave`
- `presence:subscribe`

### Proposed server messages

- `lobby:joined`
- `lobby:state`
- `lobby:player_joined`
- `lobby:player_left`
- `presence:updated`

The server must validate lobby availability, capacity, and access rules.

Presence should expose only information necessary for the social experience. Do not automatically broadcast private account details, email addresses, or session metadata.

Clients must handle players joining or leaving without assuming that their local lobby state is always current.

## 7. Parties and Invitations

### Proposed client messages

- `party:create`
- `party:invite`
- `party:accept_invite`
- `party:leave`
- `party:change_leader`

### Proposed server messages

- `party:state`
- `party:invitation_received`
- `party:invitation_result`
- `party:member_joined`
- `party:member_left`

The server validates:

- The sender's identity.
- Party membership and leadership.
- Invitation recipients.
- Invitation expiry.
- Party capacity.
- Whether the recipient can join.
- Whether the requested action is permitted in the current state.

An invitation must not grant automatic access to a match or override room capacity.

## 8. Matchmaking

### Proposed client messages

- `matchmaking:join`
- `matchmaking:cancel`

A matchmaking request may contain the selected game mode and party reference. The server must validate all options and derive party membership from trusted server state.

### Proposed server messages

- `matchmaking:queued`
- `matchmaking:updated`
- `matchmaking:cancelled`
- `match:found`
- `matchmaking:error`

A match assignment must be generated by the server. The client must not be able to choose an arbitrary match and join it without authorization.

Queue updates should not be broadcast globally if they contain private or unnecessary information.

## 9. Gameplay Input

Gameplay input is high-frequency and should use a compact payload.

An illustrative input type:

```ts
interface PlayerInput {
  sequence: number;
  movementX: number;
  movementY: number;
  lookDeltaX: number;
  lookDeltaY: number;
  buttons: number;
}
```

This is a starting point, not a finalized schema. Coordinate conventions, angle units, button encoding, and input frequency must be defined by the movement implementation.

The server validates:

- Sequence values and acceptable ordering.
- Finite numeric values and permitted ranges.
- Input frequency.
- Player life state and match phase.
- Movement and action restrictions.

Do not trust client-reported position as the authoritative position.

The server should process accepted inputs according to its simulation rules and publish authoritative state to relevant clients.

## 10. State Synchronization

The server may publish compact snapshots or use the selected framework's built-in state synchronization.

A conceptual player snapshot:

```ts
interface PlayerSnapshot {
  playerId: string;
  position: { x: number; y: number; z: number };
  rotation: { x: number; y: number; z: number };
  velocity?: { x: number; y: number; z: number };
  lifeState: "alive" | "eliminated";
}
```

The final schema must match the actual coordinate system, collision model, movement rules, and rendering requirements.

Snapshots should contain the minimum state required for clients to render and understand gameplay. Do not send private account data or unnecessary information.

Clients should interpolate remote-player updates where appropriate. If client-side prediction is implemented, include the information needed to reconcile local prediction with authoritative server state.

Define snapshot ordering and stale-update handling. A client should not replace newer state with an older snapshot.

## 11. Combat Messages

### Proposed client messages

- `weapon:fire`
- `weapon:reload`
- `weapon:change`

Payloads should identify the requested action and include only the information required by the selected weapon model.

A firing request might include a sequence number and the client's relevant input state. The server must derive the authoritative weapon state and validate the action.

### Proposed server messages

- `weapon:state`
- `combat:confirmed`
- `combat:elimination`
- `combat:rejected`

The server owns ammunition, reload state, firing cadence, damage, eliminations, and scores.

A client must not determine the final damage value or submit an unverified claim that another player was hit.

Combat messages should be designed to prevent duplicate processing and should include appropriate references to the relevant action or authoritative event.

## 12. Match Lifecycle and Results

### Proposed server messages

- `match:preparing`
- `match:started`
- `match:state`
- `match:objective_updated`
- `match:ended`
- `match:aborted`

The server owns the match phase, timer, objectives, score, and result.

A match result may contain:

- Match identifier.
- Mode and map.
- Final team or player scores.
- Outcome.
- Relevant player statistics.
- Completion timestamp.

The server must calculate or verify results before persisting them. A client-reported result must never be the source of truth for rankings or rewards.

## 13. Chat and Emotes

### Proposed client messages

- `chat:send`
- `emote:play`

### Proposed server messages

- `chat:message`
- `chat:rejected`
- `emote:played`
- `emote:rejected`

The server validates message length, permitted channel, room membership, rate limits, mute restrictions, and other applicable moderation rules.

Emotes must refer to an approved emote identifier. Do not accept arbitrary animation code or unrestricted asset URLs from clients.

V1 includes text chat and planned emotes. Voice-chat messages are outside the V1 protocol.

## 14. Player Safety

### Proposed client messages

- `player:report`
- `player:mute`
- `player:block`

### Proposed server messages

- `player:report_result`
- `player:safety_settings_updated`

Reports must be associated with the submitting player's authenticated identity. The server should validate the report category, target, and any supporting context.

The server must not reveal a reporter's private identity to the reported player through ordinary report responses.

Mute and block behavior must match the actual product policy. If a feature only affects local display, the server must not describe it as a global restriction.

## 15. Errors and Rejected Actions

Use a consistent error structure for ordinary application-level failures.

```ts
interface ProtocolError {
  code: string;
  message: string;
  requestId?: string;
  retryable?: boolean;
}
```

Possible error codes include:

- `UNAUTHENTICATED`
- `FORBIDDEN`
- `INVALID_PAYLOAD`
- `RATE_LIMITED`
- `ROOM_NOT_FOUND`
- `ROOM_FULL`
- `INVALID_GAME_STATE`
- `SESSION_EXPIRED`
- `INTERNAL_ERROR`

These are proposed identifiers and should be centralized in shared contracts.

Error messages must be useful without revealing secrets, stack traces, internal credentials, or unnecessary details about security checks.

The client should distinguish between failures it can recover from and failures that require the user to reconnect, return to the lobby, or take another action.

Do not expose detailed internal validation diagnostics to untrusted clients when they would materially help an attacker.

## 16. Rate Limits and Message Ordering

Different messages need different limits.

- Movement input: frequent, compact, and bounded.
- Weapon actions: constrained by weapon rules and input frequency.
- Chat: stricter limits to prevent spam.
- Invitations: limits per sender and recipient.
- Reports: abuse-resistant limits without blocking legitimate safety reports unnecessarily.
- Matchmaking: prevent repeated queue creation and cancellation abuse.

Use sequence numbers or equivalent mechanisms for messages that require ordering or duplicate protection.

Do not apply one arbitrary rate limit to every message category.

Rate limits should be measured and tuned to the selected networking framework and expected gameplay. They must not make ordinary play unresponsive.

## 17. Protocol Versioning

Keep protocol changes deliberate.

- Define a protocol version for independently versioned client/server contracts where appropriate.
- Validate compatibility when clients connect.
- Prefer additive changes when possible.
- Avoid changing the meaning of an existing field without a compatibility plan.
- Remove obsolete fields only after dependent clients are updated.
- Test the client and server against the same contract definitions.

If the selected framework manages compatibility differently, follow its established mechanism rather than adding redundant versioning.

## 18. Testing Requirements

Create automated tests for:

- Valid and invalid payloads.
- Unauthorized room access.
- Expired sessions.
- Duplicate action requests.
- Invalid weapon actions.
- Stale gameplay snapshots.
- Rate-limit enforcement.
- Invalid match-state transitions.
- Chat restrictions and message length.
- Protocol compatibility between client and server.

Add multiplayer integration tests using multiple connected clients. Unit tests of message schemas alone do not prove that room permissions or game-state transitions work correctly.

## 19. Implementation Order

1. Confirm the networking framework and inspect the existing code.
2. Establish shared message types and runtime validation.
3. Implement connection identity and room access.
4. Implement basic gameplay input and state synchronization.
5. Add combat actions and authoritative results.
6. Add match lifecycle and results.
7. Add lobby, party, and matchmaking messages.
8. Add chat, emotes, and safety workflows.
9. Add protocol compatibility checks, abuse tests, and performance measurements.

Implement only the messages required by the current milestone. Do not create a large unused protocol before the corresponding gameplay systems exist.

## 20. Open Decisions

- [ ] Confirm Colyseus versus NestJS + Socket.IO.
- [ ] Verify current network packages and existing message handlers.
- [ ] Finalize coordinate and angle conventions.
- [ ] Define movement-input encoding and sequence handling.
- [ ] Select the state synchronization mechanism.
- [ ] Define the initial snapshot rate and payload budget.
- [ ] Finalize combat request and result schemas.
- [ ] Define session resumption and protocol compatibility behavior.
- [ ] Confirm which contracts belong in shared packages.

## 21. Definition of Done

The protocol is ready to guide implementation when:

- Each implemented message has a documented schema.
- Runtime validation exists at the server boundary.
- Authorization and room membership are checked server-side.
- Gameplay results remain authoritative.
- Errors are consistent and do not expose sensitive internals.
- Message ordering and rate limits are defined where needed.
- Client and server contracts are tested together.
- The documented protocol matches the selected networking framework.

**Protocol rule:** Every message is a request or an observation—not permission for the client to dictate authoritative game state.