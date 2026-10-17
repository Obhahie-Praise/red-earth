# Red Earth — Multiplayer Architecture

**Path:** `docs/architecture/MULTIPLAYER.md`  
**Status:** Draft — framework decision and gameplay networking details require verification  
**Scope:** Browser-based multiplayer FPS using Three.js and a server-authoritative game model.

## 1. Purpose

This document defines how Red Earth should connect players, manage multiplayer rooms, synchronize gameplay, process combat, and handle unreliable network conditions.

The goal is to establish a practical multiplayer foundation without prematurely building a large, complicated networking system.

Red Earth is an online FPS. Multiplayer is a core requirement, not a feature to bolt on after the single-player game works.

## 2. Multiplayer Framework Decision

The latest recorded setup uses:

- **NestJS** for the game server application.
- **Socket.IO** for real-time client/server communication.
- **Three.js** for browser rendering.
- **TypeScript** for the client and server.
- Shared packages for network contracts and game types where appropriate.

**Colyseus has not been confirmed as the selected framework in the available project notes.** Its suitability should be evaluated before the multiplayer architecture is finalized.

Colyseus provides game-oriented room management and state synchronization. NestJS with Socket.IO provides more direct control but requires us to implement more of those systems ourselves.

Before committing to the full implementation, compare both options using a small multiplayer prototype. Evaluate room lifecycle, state synchronization, server-authoritative combat, reconnection, development complexity, deployment requirements, and free-tier constraints.

Do not implement two competing multiplayer frameworks.

## 3. Core Architecture

The browser is responsible for rendering and collecting player input. The server is responsible for deciding what actually happens in a match.

```text
Browser Client
├── Three.js rendering
├── Input and local camera
├── Local prediction, if needed
├── Remote-player interpolation
└── Network client
        │
        │ Real-time messages
        ▼
Authoritative Game Server
├── Connections and player sessions
├── Lobby and presence
├── Parties and invitations
├── Matchmaking
├── Match and room lifecycle
├── Movement validation
├── Combat and damage
├── Score and match results
└── Rate limits and game rules
        │
        ▼
Persistent Storage
├── Accounts and profiles
├── Progression and cosmetics
├── Match results and rankings
└── Reports and moderation records
```

This is the target logical architecture, not a claim that all these modules already exist.

The selected framework may change how rooms, state synchronization, and server modules are implemented.

## 4. Server Authority

The client must never be the final authority on competitive outcomes.

The server must validate:

- Player movement and movement limits.
- Whether a player is alive, eliminated, or allowed to act.
- Weapon ownership and equipped weapon.
- Ammunition availability and reload state.
- Fire rate and cooldowns.
- Whether an attack is valid under the game's hit-detection rules.
- Damage, eliminations, assists, and scores.
- Match timers, objectives, and completion.
- Rewards and persistent match results.

For example, the client may send a request to fire a weapon. It must not simply tell the server, "I hit this player for 100 damage," and have that accepted as fact.

The server should validate the shot using its accepted player states, weapon rules, and hit-detection method.

This does not mean every calculation must initially run on the server at an expensive simulation rate. The implementation should be measured and optimized while preserving server authority over important outcomes.

## 5. Connections and Player Identity

Each connected player needs a server-validated identity.

A connection should be associated with:

- The authenticated account, when authentication is implemented.
- A server-issued session or connection identity.
- The current room and match, if applicable.
- The player's current connection status.
- Relevant permissions and restrictions.

Never trust an account ID, moderator role, team assignment, or match result supplied by an untrusted client without validation.

A temporary guest identity may be useful during early development, but it must not become a way to impersonate another player or access protected accounts.

Connection handling must cover:

1. Initial connection.
2. Authentication or guest-session validation.
3. Joining an eligible lobby or match.
4. Unexpected disconnection.
5. Reconnection where supported.
6. Intentional departure.
7. Session expiry and cleanup.

A disconnected player must not remain indefinitely in a room or continue acting in the match.

## 6. Room and Match Model

Avoid putting every player into one enormous global room.

The architecture should distinguish the following logical spaces.

### 6.1 Lobby instances

Lobby instances allow players to discover other players, interact socially, inspect customization, and form parties.

Each lobby should have a defined capacity and lifecycle. Its capacity must be selected based on testing, not guesswork.

### 6.2 Parties and squads

A party represents players who intend to play together.

The server should validate invitations, membership changes, party leadership, and whether the party can enter a particular queue.

Party membership must not automatically grant access to a private match or privileged action.

### 6.3 Match rooms

Every active match needs an isolated game state, including:

- Match ID and mode.
- Map and match configuration.
- Participating players.
- Team assignments, where applicable.
- Player positions and gameplay state.
- Score, objectives, and remaining time.
- Match phase and final results.

A player must not be able to subscribe to or modify another match's private state merely by supplying its room ID.

### 6.4 Private matches

Private 1v1 matches are a planned social feature, subject to implementation planning.

The server should validate invitations, participants, match settings, and access to the private room. Private matches must not contaminate public matchmaking or competitive rankings unless the ranking rules explicitly allow it.

## 7. Network Messages and Shared Contracts

Client and server messages must use explicit, typed contracts.

Create or reuse a shared package for network message types and validation schemas after verifying the existing monorepo structure.

Potential message categories include:

| Category | Examples |
|---|---|
| Connection | `session:resume`, `session:ready` |
| Lobby | `lobby:join`, `lobby:leave`, `lobby:state` |
| Party | `party:invite`, `party:join`, `party:leave` |
| Matchmaking | `matchmaking:join`, `matchmaking:cancel`, `match:found` |
| Gameplay | `input:command`, `player:snapshot`, `player:action` |
| Combat | `weapon:fire`, `weapon:reload`, `combat:result` |
| Match | `match:started`, `match:state`, `match:ended` |
| Social | `chat:send`, `chat:message`, `player:emote` |
| Safety | `player:report`, `player:mute`, `player:block` |

These names are illustrative contract proposals, not existing event names.

For every message, define its payload, sender, recipient scope, validation rules, and expected response. Do not create duplicate event names that perform the same operation.

Validate network payloads at runtime. TypeScript types alone do not protect the server from malformed messages.

Use consistent error responses for invalid actions, unauthorized access, expired sessions, unavailable rooms, and rate limits.

## 8. Movement Synchronization

Movement synchronization must balance responsiveness, fairness, and bandwidth.

### Client responsibilities

The client should:

- Read movement and look input.
- Send compact input commands rather than uploading entire game state.
- Render the local player's view.
- Render remote players using received authoritative state.
- Predict local movement only if the implementation requires it.
- Correct local prediction when the server disagrees.

### Server responsibilities

The server should:

- Process accepted input.
- Apply movement rules and limits.
- Validate collisions and legal movement.
- Maintain the authoritative player state.
- Publish updates to the relevant players.
- Reject impossible or suspicious movement.

### Remote-player interpolation

Remote players should normally be rendered between recent server updates instead of snapping directly to every received position.

### Local-player prediction

Prediction may make movement feel responsive, but it introduces reconciliation complexity. If used, the client must be able to correct itself to match the server's accepted state.

Begin with the simplest movement synchronization that feels acceptable. Add prediction and reconciliation when testing demonstrates the need.

Do not implement a complex custom prediction system before the basic multiplayer loop works.

## 9. Combat and Hit Detection

Combat is especially sensitive to latency and cheating.

A proposed firing flow:

1. The client sends a firing command with the necessary input and sequence information.
2. The server checks the player's state, weapon, ammunition, cooldown, and firing rules.
3. The server evaluates the shot using the selected hit-detection approach.
4. The server applies valid damage and updates eliminations, scores, and other match state.
5. The server sends the resulting state or events to affected players.

The protocol must account for duplicate messages, delayed packets, rapid repeated firing, reload timing, and players disconnecting during combat.

### Hit-detection decision

The initial implementation must explicitly choose a hit-detection model. For example, a server-side raycast may be suitable for hitscan weapons, while projectile weapons require their own server-validated simulation.

Lag compensation, if added, must use bounded historical state and strict limits. It must not allow a client to request arbitrary historical hit results.

The final choice depends on the weapon design and the movement/collision implementation.

## 10. Tick Rate and State Updates

The server simulation rate and network update rate are separate decisions.

- **Simulation rate:** how often the server advances gameplay.
- **Snapshot rate:** how often updated state is sent to clients.
- **Rendering rate:** how often the browser draws frames.

Do not assume all three should run at the same frequency.

Choose initial rates through a prototype and test them on realistic connections. Measure CPU usage, message sizes, bandwidth, input responsiveness, and state correction.

Use compact snapshots for frequently changing state. Send less frequent events for things such as match transitions, eliminations, and persistent progression.

Avoid broadcasting every player's entire state to every connected player when only a subset needs it. Interest management can be introduced when map size and player count justify it.

## 11. Latency and Unreliable Networks

The game must remain understandable when connections are slow or unstable.

Handle:

- Delayed and out-of-order state updates.
- Temporary packet loss.
- Short disconnections.
- Repeated commands.
- Reconnection during a match.
- Players leaving during an active match.
- Server-side timeouts.

Use sequence numbers or equivalent ordering mechanisms where needed. Do not assume that a received client command is fresh merely because it arrived.

Reconnection policy should define whether a player can resume a match, how long the server retains their slot, and what happens if they do not return.

The policy may differ by mode. A disconnect in a small team match should not automatically receive the same treatment as a disconnect in a future battle royale match.

## 12. Matchmaking and Match Lifecycle

The match lifecycle should use explicit states rather than loosely related booleans.

Proposed states:

`waiting → preparing → in_progress → completed`

A match may also transition to a cancelled or aborted state when appropriate.

The server must control transitions, validate readiness, assign teams, start timers, and determine match completion.

Matchmaking should consider:

- Selected mode.
- Party size.
- Map and mode compatibility.
- Available server capacity.
- Queue time.
- Ranking constraints, once competitive matchmaking is implemented.

Begin with straightforward queueing and capacity rules. More advanced matchmaking should follow real player population and queue-time data.

## 13. Chat, Presence, and Emotes

Red Earth V1 includes text chat, social presence, and planned emotes. Voice chat is not part of V1.

Chat should be scoped to appropriate destinations, such as lobby, squad, and team. The server must enforce room membership and any mute, block, or moderation restrictions.

Presence should distinguish between being online, being in a lobby, and being in a match where useful.

Emotes should be server-validated actions with rate limits. They should not interrupt gameplay in a way that creates unfair advantages.

Invitations and presence notifications must be sent only to authorized recipients.

## 14. Security and Abuse Prevention

All client input is untrusted.

At minimum:

- Validate payload shapes and sizes.
- Limit message frequency.
- Enforce room membership and permissions.
- Reject invalid state transitions.
- Validate weapon actions and movement.
- Prevent duplicate or replayed actions where relevant.
- Restrict chat and invitation spam.
- Avoid broadcasting private session data.
- Record meaningful security and moderation events.
- Keep secrets and privileged credentials out of browser code.

Do not rely on hiding buttons in the client to enforce permissions.

Anti-cheat will be an ongoing process, not a single feature. Server authority, sensible validation, telemetry, reporting, and moderation reduce risk but cannot guarantee the absence of cheating.

## 15. Scaling and Deployment

Do not promise a specific concurrent-player capacity before testing.

The practical capacity depends on the simulation workload, room sizes, snapshot frequency, message volume, network conditions, and deployment environment.

Start with isolated match instances and measure performance under realistic load.

Before running multiple server instances, determine how matchmaking, presence, room ownership, and reconnection will work across instances. A shared message broker or coordination layer may eventually be required, depending on the framework and deployment model.

The deployment platform must support the selected real-time transport and its connection lifecycle. Do not assume that a platform suited to ordinary HTTP requests automatically supports persistent game connections.

Keep the initial deployment simple and cost-conscious. Evaluate actual free-tier limits and connection constraints before choosing production infrastructure.

## 16. Observability and Testing

Record operational metrics without logging sensitive session credentials.

Useful metrics include:

- Active connections and players per room.
- Match starts, completions, and cancellations.
- Server tick duration and missed ticks.
- Snapshot size and send frequency.
- Connection failures and reconnect attempts.
- Message validation failures and rate limits.
- Matchmaking wait time.
- Server CPU and memory usage.
- Unexpected room termination.

Test at several levels:

1. Unit tests for movement, combat, and match rules.
2. Contract tests for message validation.
3. Integration tests for joining, leaving, and reconnecting.
4. Multiplayer tests with multiple browser clients.
5. Load tests for room capacity and server performance.
6. Adversarial tests for forged damage, invalid movement, repeated firing, and unauthorized room access.

A successful local test with two players does not establish production capacity.

## 17. Implementation Phases

### Phase 1 — Framework verification

Build a small prototype comparing the currently recorded NestJS + Socket.IO approach with Colyseus.

Test room creation, state synchronization, server-side movement, a basic weapon action, disconnect handling, and local development experience.

Document the selected framework and the reasons for the choice before implementing the full multiplayer architecture.

### Phase 2 — Basic multiplayer loop

- Connect multiple clients.
- Create and join a room.
- Spawn players.
- Synchronize movement.
- Handle disconnects and room cleanup.
- Validate messages.

### Phase 3 — Authoritative combat

- Implement weapon state and firing rules.
- Validate hits and damage on the server.
- Synchronize eliminations and scores.
- Complete a basic match lifecycle.

### Phase 4 — Social and matchmaking

- Add lobby instances and presence.
- Implement parties and invitations.
- Add text chat and moderation controls.
- Implement basic matchmaking.
- Support private matches if the scope and testing permit.

### Phase 5 — Performance and reliability

- Test realistic player counts.
- Tune simulation and snapshot rates.
- Add reconnection behavior.
- Profile server and browser performance.
- Validate deployment constraints.
- Add monitoring and abuse protection.

Additional game modes and battle royale scale should be built on top of a tested foundation, not assumed to work merely because a small team match works.

## 18. Decisions Still Open

The following decisions must be resolved through repository inspection, prototyping, or explicit product decisions:

- [ ] Confirm Colyseus versus NestJS + Socket.IO.
- [ ] Choose the initial match and lobby capacities.
- [ ] Set initial simulation and snapshot rates.
- [ ] Define movement and collision simulation.
- [ ] Choose hit detection for each weapon class.
- [ ] Decide whether and how to implement lag compensation.
- [ ] Define disconnect and reconnection rules per mode.
- [ ] Establish initial matchmaking rules.
- [ ] Choose the production deployment platform.
- [ ] Determine when multi-instance coordination becomes necessary.
- [ ] Define battle royale player capacity based on measured performance.
- [ ] Confirm how persistent profiles, rankings, and match results are stored.

## 19. Completion Criteria

This architecture is ready to guide full implementation when:

- The multiplayer framework decision is documented.
- Two or more browser clients can join the same match.
- Movement is synchronized without unacceptable snapping.
- The server controls combat outcomes and match results.
- Invalid client commands are rejected.
- Disconnects and room cleanup work reliably.
- Message contracts are typed and runtime-validated.
- Basic performance and abuse tests exist.
- Deployment assumptions have been verified rather than guessed.

**Core rule:** Red Earth must feel responsive to the player, but the server remains the authority on competitive gameplay.