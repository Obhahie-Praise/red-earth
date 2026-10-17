# Red Earth — Feature Specification

**Status:** Draft  
**Purpose:** Define the feature inventory, scope boundaries, dependencies and completion expectations for Red Earth.

## 1. Purpose

This document translates the Red Earth product vision and game design into implementable feature areas.

It serves as a reference for planning, development and testing. It must be read alongside:

- `VISION.md` for the product direction.
- `GAME_DESIGN.md` for gameplay and player experience.
- The architecture documents for technical constraints.
- `docs/roadmap/ROADMAP.md` for implementation order.
- `docs/roadmap/PROGRESS.md` for actual implementation status.

A feature listed here is not automatically implemented. Completion must be supported by working code and appropriate verification.

## 2. Scope Categories

Features are organized into three categories.

- **V1 Core:** Required to establish the initial playable and socially functional release.
- **V1 Content Target:** Content intended for the full V1 scope, subject to production and performance validation.
- **Later:** Features that can follow after the core experience is reliable.

The roadmap determines sequencing. It must not silently remove agreed product requirements; any scope changes should be recorded in `DECISIONS.md`.

## 3. Feature Inventory

| ID | Feature | Scope | Priority |
|---|---|---|---|
| RE-001 | Browser game shell and startup | V1 Core | Critical |
| RE-002 | Responsive non-gameplay interface | V1 Core | High |
| RE-003 | Account and player identity | V1 Core | High |
| RE-004 | 3D rendering and scene management | V1 Core | Critical |
| RE-005 | Player movement and camera | V1 Core | Critical |
| RE-006 | Weapon handling and combat | V1 Core | Critical |
| RE-007 | Authoritative multiplayer server | V1 Core | Critical |
| RE-008 | Match lifecycle and results | V1 Core | Critical |
| RE-009 | First playable map | V1 Core | Critical |
| RE-010 | Team Deathmatch prototype | V1 Core candidate | High |
| RE-011 | Additional game modes | V1 Content Target | High |
| RE-012 | Five-map content target | V1 Content Target | High |
| RE-013 | Weapon roster | V1 Content Target | High |
| RE-014 | Character roster | V1 Content Target | High |
| RE-015 | Character and weapon customization | V1 Core, limited initial scope | High |
| RE-016 | Lobby and 3D character preview | V1 Core | High |
| RE-017 | Friends, invitations and squads | V1 Core | High |
| RE-018 | Private 1v1 challenges | V1 Core candidate | Medium |
| RE-019 | Lobby, squad and team text chat | V1 Core | High |
| RE-020 | Starter emotes | V1 Core, limited set | Medium |
| RE-021 | Player mute, block and reporting | V1 Core | High |
| RE-022 | Administrative moderation tools | V1 Core | High |
| RE-023 | Operational analytics | V1 Core, essential metrics | High |
| RE-024 | Ranking and leaderboards | V1 Content Target | High |
| RE-025 | Currency and cosmetic rewards | V1 Content Target | Medium |
| RE-026 | Animated loading and progress feedback | V1 Core | High |
| RE-027 | Performance monitoring and optimization | V1 Core | Critical |
| RE-028 | Automated tests and CI checks | V1 Core | High |
| RE-029 | Mobile FPS touch controls | Later unless explicitly approved | Deferred |
| RE-030 | Voice chat | Later | Deferred |
| RE-031 | Advanced cosmetic economy | Later | Deferred |
| RE-032 | Advanced competitive and social features | Later | Deferred |

The roadmap may refine priorities as technical feasibility is established. Changes affecting agreed requirements must be documented.

## 4. Platform and Startup

### RE-001: Browser Game Shell and Startup

**Goal:** Let players open Red Earth in a supported browser and reach the appropriate entry point.

Requirements:

- Provide a coherent application shell.
- Handle startup failures without leaving a blank screen.
- Display meaningful loading feedback.
- Handle unsupported browser capabilities gracefully.
- Avoid requiring a standalone game installation.
- Separate the web interface from the 3D game runtime where appropriate.

Acceptance criteria:

- A fresh visit displays a usable startup state.
- Startup failures produce a visible, understandable error.
- A successful startup reaches the appropriate screen.
- Repeated navigation does not create duplicate game runtimes or connections.

### RE-002: Responsive Non-gameplay Interface

Requirements:

- Support desktop layouts as the primary design target.
- Adapt relevant interface screens to smaller displays.
- Prevent clipped navigation, inaccessible controls and unusable forms.
- Keep full FPS combat desktop-first in V1.

Acceptance criteria:

- Supported interface pages remain usable at agreed desktop, tablet and mobile widths.
- Responsive layout does not imply that touch-based combat is complete.

## 5. Identity and Accounts

### RE-003: Account and Player Identity

Requirements:

- Provide a persistent player identity.
- Associate social features and progression with the correct account.
- Validate permissions and ownership on the server.
- Avoid trusting player identifiers supplied by the client without authorization checks.

Account registration, authentication providers, account recovery and identity verification details must be finalized in the architecture and security documentation.

Acceptance criteria:

- Authenticated actions resolve to the correct server-verified player.
- Players cannot impersonate another account by changing client-side identifiers.
- Relevant account state persists across supported sessions.

## 6. Rendering and Combat

### RE-004: 3D Rendering and Scene Management

Requirements:

- Use Three.js for the 3D experience.
- Support scene loading, camera management, lighting, models and materials.
- Manage GPU resources and dispose of unused resources.
- Use optimized assets and sensible quality settings.
- Avoid unnecessary allocations and expensive per-frame operations.

Acceptance criteria:

- Scenes load and transition without creating duplicate render loops.
- Unused resources are released where appropriate.
- Performance is measured against an agreed baseline.

### RE-005: Player Movement and Camera

Requirements:

- Implement responsive first-person movement.
- Support mouse look and agreed desktop controls.
- Provide appropriate collision and movement validation.
- Synchronize relevant player movement through the authoritative multiplayer architecture.

Acceptance criteria:

- Players cannot move through blocking geometry under normal conditions.
- Movement remains responsive under tested network conditions.
- Invalid movement updates are rejected or corrected by the server where required.

### RE-006: Weapon Handling and Combat

Requirements:

- Support firing, reloading, ammunition state and weapon switching.
- Implement distinct weapon categories.
- Validate firing cadence, ammunition, hit results and damage.
- Provide visual and audio feedback when appropriate.
- Keep authoritative combat outcomes on the server.

The full weapon roster is a content target, not a requirement for the first combat prototype.

Acceptance criteria:

- Valid weapon actions work as intended.
- Invalid firing and reload actions are rejected.
- Damage and eliminations cannot be determined solely by untrusted client messages.

## 7. Multiplayer and Matches

### RE-007: Authoritative Multiplayer Server

Requirements:

- Establish authenticated or otherwise validated player sessions.
- Manage player presence and match membership.
- Validate client messages and their payloads.
- Enforce important gameplay rules on the server.
- Handle disconnections and reconnect attempts safely.
- Apply rate limits to abuse-prone actions.

Acceptance criteria:

- Two or more clients can exchange the required gameplay state.
- Invalid payloads do not crash the server.
- A client cannot independently award itself authoritative match results.

### RE-008: Match Lifecycle and Results

Requirements:

- Support match creation, joining and starting.
- Track match state and participant membership.
- Enforce the selected mode's completion rules.
- Record validated results.
- Clean up finished matches and disconnected participants.

Acceptance criteria:

- Matches transition through valid states.
- Invalid state transitions are rejected.
- Results are consistent for participating clients.
- Finished matches do not remain active indefinitely.

### RE-009: First Playable Map

Requirements:

- Provide one complete, playable environment.
- Include collision, spawn points and a defined combat layout.
- Optimize geometry, textures and lighting.
- Test map balance and traversal.

Acceptance criteria:

- Players can spawn, move, fight and complete a match on the map.
- Spawn positions and collision boundaries are validated.
- The map meets the initial performance target.

### RE-010 and RE-011: Game Modes

The first mode should be selected based on the simplest route to a reliable multiplayer prototype. Team Deathmatch is the current recommended candidate.

Each implemented mode must define:

- Team or individual participation.
- Match start conditions.
- Spawn and respawn rules.
- Scoring.
- Victory and completion conditions.
- Disconnect handling.
- Result calculation.

Battle royale requires separate validation of map size, player count, loot distribution, safe-zone mechanics, server load and matchmaking.

### RE-012: Map Content Target

The intended content target is at least five maps.

Each map must have a documented layout, intended modes, collision, spawn points, performance budget and validation checklist.

Maps should be added incrementally after the rendering and combat pipeline is stable.

## 8. Weapons and Characters

### RE-013: Weapon Roster

Target roster:

- Three assault rifles.
- Three SMGs.
- Two shotguns.
- Grenades and flashbangs.

Requirements:

- Weapon statistics must be explicit and server-validated.
- Weapon classes must have distinguishable gameplay roles.
- Models and effects must meet the performance budget.
- Weapon skins must not alter competitive statistics.

### RE-014: Character Roster

The intended content target is at least five characters.

Requirements:

- Character assets must meet agreed quality and performance budgets.
- Models and animations should be reusable where practical.
- Character selection must be reflected consistently in supported views.
- Cosmetic differences must preserve gameplay readability.

### RE-015: Customization

Requirements:

- Show owned and currently equipped cosmetics.
- Preview supported character and weapon appearances.
- Equip only items the player is authorized to use.
- Persist validated selections.
- Handle missing or failed cosmetic assets gracefully.

The initial cosmetic inventory may be small, but the ownership and equipment model should not require a complete redesign when new items are introduced.

## 9. Lobby and Social Systems

### RE-016: Lobby and 3D Character Preview

Requirements:

- Provide a cinematic 3D character presentation.
- Display the equipped weapon and supported cosmetic appearance.
- Offer straightforward access to Play, Loadout, Customize and Social.
- Keep important actions accessible.
- Support graceful fallback if preview assets fail to load.

### RE-017: Friends, Invitations and Squads

Requirements:

- Display relevant player presence.
- Send and accept invitations.
- Create and leave squads.
- Track squad membership and leadership.
- Validate permissions for party management.
- Handle expired invitations and disconnected players.

The exact maximum squad size and social discovery rules remain open decisions.

### RE-018: Private 1v1 Challenges

Requirements:

- Send, accept and decline duel challenges.
- Validate participant eligibility.
- Create an appropriately configured private match.
- Apply normal server-authoritative combat rules.
- Return both players to an appropriate post-match state.

Ranked duels and advanced custom rules are outside the initial requirement unless separately approved.

### RE-019: Text Chat

Requirements:

- Support the agreed lobby, squad and team chat channels.
- Validate and rate-limit messages.
- Associate messages with verified player identities.
- Support moderation and reporting.
- Handle disconnects and delivery failures.

Voice chat is excluded from V1.

### RE-020: Emotes

Requirements:

- Provide a small starter set of emotes.
- Allow players to select and trigger supported emotes.
- Use reusable animations where possible.
- Prevent excessive activation through cooldowns or rate limits.
- Avoid interfering with essential gameplay controls.
- Handle unavailable assets gracefully.

## 10. Safety and Administration

### RE-021: Player Safety and Reporting

Requirements:

- Allow reports against players and relevant chat messages.
- Offer clear report categories.
- Capture necessary contextual identifiers.
- Support mute and block actions.
- Prevent repeated report submissions from overwhelming the system.
- Keep reports and moderation metadata access-controlled.

### RE-022: Administrative Moderation

Requirements:

- Restrict administrative access through server-enforced permissions.
- Allow authorized administrators to find players and review reports.
- Support temporary and permanent bans with reasons.
- Track moderation history and administrative actions.
- Provide appropriate safeguards against accidental or unauthorized actions.
- Define a process for ban expiry, appeals and reversals.

Administrative actions must be auditable.

### RE-023: Operational Analytics

Initial metrics should prioritize service health and actual usage:

- Registered accounts.
- Daily active players.
- Concurrent players.
- Matches started and completed.
- Average match duration.
- Disconnections and connection failures.
- Pending and resolved reports.
- Moderation actions.

Metrics must have clear definitions. For example, daily active players should use a defined time window and a consistent rule for what counts as activity.

Collect only information needed to operate, improve and moderate the game. More advanced retention and engagement analytics can follow later.

## 11. Loading and Performance

### RE-026: Loading Feedback

Requirements:

- Provide a branded initial loading experience.
- Show progress where measurable.
- Use honest indeterminate feedback where progress cannot be measured.
- Display relevant map or connection states.
- Show retry options for recoverable failures.
- Avoid unnecessary full-screen transitions for small operations.

### RE-027: Performance and Monitoring

Requirements:

- Establish target frame rates and minimum supported hardware.
- Track loading time and major failure categories.
- Optimize model complexity, textures, draw calls and scene resources.
- Test representative low-spec devices.
- Test behavior under latency, packet loss and disconnections.
- Avoid unnecessary paid infrastructure and services until justified.

The actual frame-rate, memory, latency and player-count targets must be defined through technical prototypes and measurements.

## 12. Testing and Delivery

### RE-028: Tests and CI

Requirements:

- Type-check the relevant packages and applications.
- Run linting and automated tests.
- Test important server validation rules.
- Test match lifecycle and disconnection handling.
- Verify shared contracts between client and server.
- Include regression coverage for fixed defects.
- Document any tests that require a running browser, server or multiple clients.

A successful build is not proof that multiplayer gameplay works. The roadmap must include real integration tests with multiple clients.

## 13. Later Features

The following are explicitly deferred from the initial V1 requirements:

- Voice chat.
- Full mobile FPS touch controls.
- A complex cosmetic marketplace or economy.
- Advanced competitive systems beyond the initial agreed ranking scope.
- Advanced social features not required for squads and text communication.

Deferral does not mean these features are permanently rejected. They require a separate scope decision before implementation.

## 14. Dependencies and Recommended Order

The principal implementation dependencies are:

1. Repository and development environment verification.
2. Application shell and game runtime startup.
3. Rendering, camera, movement and collision.
4. Server connection and shared multiplayer contracts.
5. Authoritative combat and match lifecycle.
6. First playable map and first playable game mode.
7. Multiplayer testing and performance validation.
8. Lobby, player identity and customization.
9. Friends, squads, invitations and text chat.
10. Reporting, moderation and operational analytics.
11. Additional maps, modes, weapons and characters.
12. Ranking, progression and cosmetic rewards.

This is a dependency-oriented recommendation, not permission to ignore agreed product requirements. The roadmap must reconcile this sequence with the full V1 content target.

## 15. Feature Completion Standard

A feature is complete only when:

- Its requirements are implemented.
- Relevant automated tests pass.
- Its user-facing behavior has been verified.
- Error and disconnection states have been considered.
- Security and server-authority requirements have been checked.
- Documentation and the progress tracker are updated.
- Its status is supported by evidence rather than assumption.

No feature should be marked complete solely because its UI exists or its code compiles.

## 16. Open Decisions

The following must be settled or explicitly deferred:

- Exact minimum browser and hardware requirements.
- Initial player counts per mode.
- Exact V1 mode list and map-to-mode assignments.
- Squad size and party permissions.
- Combat statistics and balance targets.
- Ranking and progression formulas.
- Currency earning and spending rules.
- Initial cosmetic ownership and unlock rules.
- Starter emote selection.
- Analytics retention and privacy policy.
- The exact mobile social features supported at launch.

Record material decisions in `docs/roadmap/DECISIONS.md` and update the roadmap and implementation status when they change.