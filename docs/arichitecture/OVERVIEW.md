# Red Earth — Architecture Overview

**Status:** Draft  
**Purpose:** Define the system boundaries, responsibilities and communication flows for Red Earth.

## 1. Overview

Red Earth is a browser-first multiplayer FPS built around a 3D web client and an authoritative multiplayer server.

The architecture must support real-time gameplay, social features, persistent player data and future expansion without making the initial project unnecessarily complex or expensive to operate.

The intended foundation is a pnpm/Turborepo monorepo containing the web application, game server and shared packages where appropriate.

This document describes the intended architecture. It does not assert that all components or features have already been implemented.

## 2. Architectural Goals

The architecture should:

- Support real-time multiplayer FPS gameplay.
- Keep authoritative game rules on the server.
- Deliver a performant Three.js experience through a browser.
- Separate gameplay, rendering, social systems and persistent data.
- Reuse shared types and contracts between client and server.
- Validate all untrusted client input.
- Make testing and local development straightforward.
- Keep infrastructure costs low during early development.
- Allow additional game modes, maps and cosmetic content without major architectural rewrites.

## 3. System Components

### 3.1 Web Application — `apps/web`

**Intended stack:** Next.js, React, TypeScript and Three.js.

The web application is responsible for:

- Application navigation and interface rendering.
- Account and player-facing pages.
- Lobby, customization and social interfaces.
- Rendering the 3D game world.
- Collecting player input.
- Sending gameplay and social actions to the server.
- Displaying synchronized player and match state.
- Presenting loading, connection and error states.

The web application must not independently determine authoritative competitive outcomes.

React should manage application-level state and interface composition. The Three.js runtime should manage the game scene, rendering loop, camera, game objects and rendering-specific resources.

The two should communicate through clearly defined interfaces rather than duplicating state or creating unnecessary rendering loops.

### 3.2 Multiplayer Server — `apps/server`

**Intended stack:** NestJS, TypeScript and Socket.IO.

The server is responsible for:

- Establishing and validating player sessions.
- Managing real-time connections and player presence.
- Managing match membership and lifecycle.
- Validating player actions.
- Maintaining authoritative gameplay state.
- Validating combat actions and calculating outcomes.
- Managing squads and invitations.
- Routing permitted chat messages.
- Validating cosmetic ownership and equipment changes.
- Recording match results and relevant player activity.
- Enforcing rate limits and moderation rules.

The server must treat the browser as an untrusted client.

Important game rules must be enforced server-side even when the client also performs local checks for responsiveness.

### 3.3 Shared Packages — `packages/`

Shared packages should contain code genuinely needed by multiple applications.

Potential responsibilities include:

- Shared TypeScript types.
- Client-server event contracts.
- Runtime validation schemas.
- Shared constants and identifiers.
- Small, deterministic utility functions.

Do not create packages solely to make the monorepo look more sophisticated. Each package should have a clear purpose and dependency boundary.

The actual package structure must be confirmed against the repository before it is documented as existing.

### 3.4 Persistent Storage

Red Earth will require persistent storage for data such as:

- Player accounts and profiles.
- Cosmetic ownership and equipped items.
- Friends and relevant social relationships.
- Match results and progression.
- Reports and moderation history.
- Administrative audit records.

The database and persistence strategy must be selected or verified based on the existing repository, deployment constraints, expected workload and cost.

Do not introduce a database, ORM or managed service solely because it is common in similar projects. Document the chosen provider, schema, migrations, backups and operational requirements once verified.

### 3.5 Asset Delivery

Game assets may include:

- Character models and animations.
- Weapon models and skins.
- Map geometry and environment assets.
- Textures and materials.
- Sound effects and music.
- Interface artwork and loading-screen media.

Large assets should be optimized and delivered separately from application code where practical.

The asset-delivery strategy should support caching, progressive loading and versioned asset references. The exact storage and delivery provider remains dependent on the project's deployment decisions.

## 4. High-Level Architecture

The intended logical flow is:

```text
Player's Browser
│
├── Next.js Application
│   ├── Account and Profile UI
│   ├── Lobby and Social UI
│   ├── Customization UI
│   └── Three.js Game Runtime
│       ├── Rendering and Scene Management
│       ├── Camera and Input
│       ├── Local Presentation State
│       └── Network State Adapter
│
└── Socket.IO Client
        │
        ▼
NestJS Multiplayer Server
│
├── Connection and Session Management
├── Player and Squad Services
├── Matchmaking and Match Lifecycle
├── Authoritative Gameplay
├── Combat Validation
├── Chat and Moderation
└── Persistence and Results
        │
        ▼
Persistent Storage
```

This is a logical architecture, not a requirement to deploy every module as a separate service.

The initial implementation should favor a well-structured server application over premature microservice separation.

## 5. Communication Boundaries

### 5.1 HTTP

HTTP is appropriate for operations such as:

- Loading account or profile information.
- Retrieving owned cosmetics.
- Updating supported account settings.
- Administrative dashboard queries.
- Retrieving match history or other persistent records.

Each endpoint must enforce authentication and authorization where required.

### 5.2 Socket.IO

Real-time communication is appropriate for:

- Player presence.
- Match creation and joining.
- Player movement updates.
- Combat action requests.
- Match state updates.
- Squad invitations and membership changes.
- Lobby and team chat.
- Match lifecycle events.

Events must use explicit contracts and server-side validation. Avoid accepting arbitrary event names or unbounded payloads from clients.

### 5.3 Persistent Data

Persistent data should be written through server-controlled services.

Clients must not directly determine account balances, cosmetic ownership, official scores, bans or competitive rankings.

Frequently updated transient gameplay state should not automatically be written to the database on every network tick. Persistence should be designed around durability requirements and operational cost.

## 6. Authoritative Multiplayer Model

The server is the source of truth for important competitive state.

The browser handles input, rendering and immediate visual feedback. The server validates actions and determines authoritative results.

Examples include:

- Whether a player is permitted to join a match.
- Whether a weapon can fire.
- Whether ammunition is available.
- Whether a reload is valid.
- Whether a hit and its damage are valid.
- Whether a player has been eliminated.
- Whether a match has ended.
- Which results and rewards are official.

Client-side prediction may be introduced to make movement and actions feel responsive. When predicted state conflicts with authoritative state, the client must reconcile it appropriately.

The initial network model should be selected through a working multiplayer prototype. Update frequency, interpolation, prediction and reconciliation should be measured rather than guessed.

## 7. State Ownership

Each type of state should have a clear owner.

| State | Primary authority |
|---|---|
| Camera and rendering resources | Browser |
| Raw keyboard and mouse input | Browser |
| Local visual effects | Browser |
| Authoritative player position | Server |
| Match membership and phase | Server |
| Ammunition and combat outcomes | Server |
| Official match scores | Server |
| Player profile persistence | Server and database |
| Cosmetic ownership | Server and database |
| Equipped cosmetic selection | Server-validated persistent state |
| Chat moderation decisions | Server |
| Administrative permissions | Server |
| Loading presentation | Browser |

The client may maintain temporary copies of server state for rendering, but those copies must not be treated as authoritative.

## 8. Performance and Resource Management

The architecture should account for browser rendering and network limitations from the beginning.

### Rendering

- Maintain a single controlled render loop per active game runtime.
- Avoid unnecessary React rerenders for high-frequency gameplay updates.
- Reuse geometries, materials and resources where appropriate.
- Dispose of resources when scenes or assets are no longer needed.
- Use sensible lighting, texture sizes, draw-call budgets and levels of detail.
- Provide graceful behavior when WebGL or required capabilities are unavailable.

### Networking

- Avoid transmitting unchanged state unnecessarily.
- Separate high-frequency gameplay updates from infrequent social events.
- Validate payload sizes and rates.
- Handle packet loss, latency and disconnections.
- Measure bandwidth and server processing cost under realistic test conditions.

### Asset loading

- Load essential assets first.
- Defer optional cosmetics and nonessential detail where practical.
- Show honest loading progress when measurable.
- Cache versioned assets where appropriate.
- Recover gracefully from failed asset requests.

## 9. Security Boundaries

The browser must be treated as an untrusted environment.

The server must validate:

- Identity and session validity.
- Match and squad membership.
- Payload shape and size.
- Action permissions.
- Combat rules.
- Cosmetic ownership.
- Chat limits.
- Report submission rules.
- Administrative authorization.

Secrets must never be embedded in browser bundles. Administrative operations must not rely on hidden frontend controls as their only protection.

Additional security requirements belong in `docs/architecture/SECURITY.md`.

## 10. Testing Strategy

Testing should cover individual modules and the interactions between the web client and server.

Required testing areas include:

- Unit tests for deterministic gameplay and validation logic.
- Contract tests for shared event payloads.
- Integration tests for match lifecycle and player sessions.
- Multiplayer tests using multiple clients.
- Reconnection and disconnection scenarios.
- Security tests for invalid actions and unauthorized access.
- Rendering and performance tests on representative hardware.

A successful build or a single-client demo is not sufficient evidence that multiplayer works correctly.

## 11. Deployment and Operations

Deployment must preserve the separation between the browser application and the multiplayer server.

Before selecting production infrastructure, verify:

- Whether the host supports long-lived WebSocket connections.
- Whether persistent server processes are available.
- Connection and concurrent-player limits.
- Regional latency for the intended player base.
- Database availability and connection limits.
- Bandwidth and outbound traffic costs.
- Logging and monitoring availability.
- Free-tier restrictions and upgrade requirements.

Do not assume that a static hosting provider can run a persistent multiplayer server. Do not introduce paid services before evaluating viable free or low-cost options.

The final deployment topology must be recorded after the actual hosting environment is selected.

## 12. Architecture Evolution

The initial implementation should favor clear module boundaries within a manageable application.

Separate services should be introduced only when justified by concrete needs such as independent scaling, operational isolation or deployment constraints.

Future growth may require dedicated services for matchmaking, game simulation, chat or analytics. Those are potential evolution paths, not current requirements.

Architecture changes must preserve authoritative gameplay, maintainability and reasonable operating costs.

## 13. Definition of Completion

This architecture is sufficiently defined for implementation when:

- The actual monorepo structure has been verified.
- Responsibilities of the web client and server are clear.
- Shared contracts have an explicit location and validation strategy.
- State ownership and server authority are documented.
- Persistence and asset-delivery choices are confirmed.
- The first multiplayer mode has a measurable networking strategy.
- Deployment assumptions have been validated.
- Relevant decisions are recorded in `docs/roadmap/DECISIONS.md`.