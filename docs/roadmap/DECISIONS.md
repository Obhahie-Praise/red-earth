# Architecture & Product Decisions

**Project:** Red Earth  
**Path:** `docs/roadmaps/DECISIONS.md`  
**Status:** Living document

## 1. Purpose

This document records important product, gameplay, architecture, and development decisions for Red Earth.

It exists to:

- Preserve decisions across development sessions.
- Prevent coding agents from contradicting established requirements.
- Separate confirmed decisions from proposals and unresolved questions.
- Explain why important choices were made.
- Track changes when an earlier decision is deliberately replaced.

This document complements the product, architecture, and development documentation. It does not replace their detailed specifications.

## 2. Decision Status

Every decision should have one of these statuses:

- **Accepted:** Explicitly agreed upon and should be followed.
- **Proposed:** Recommended but not yet explicitly approved.
- **Open:** Requires a decision before the affected implementation can be finalized.
- **Superseded:** Replaced by a newer decision. Retain the record for historical context.

Do not mark a proposal as accepted simply because it appears in another draft document.

## 3. Accepted Decisions

### DEC-001 — Browser-first game

**Status:** Accepted

Red Earth will be a browser-first multiplayer FPS. Players should be able to access the game through a web browser rather than needing to install a standalone desktop game.

**Implications:**
- The web client is a core part of the product.
- Download size, loading time, and browser performance matter.
- Large assets should be loaded and managed carefully.
- The architecture must support real-time multiplayer in the browser.

### DEC-002 — Nigerian and Edo-inspired identity

**Status:** Accepted

The game will draw inspiration from Nigeria, with particular attention to Edo heritage and Benin City.

**Implications:**
- Environments, maps, visual details, and world-building should establish a distinctive regional identity.
- Cultural inspiration should be handled deliberately rather than reduced to generic African imagery.
- Performance and competitive readability must remain important when designing detailed environments.

### DEC-003 — Red Earth visual identity

**Status:** Accepted

The working title is **Red Earth**. The visual direction is stylized realism with painterly qualities, using red-clay terrain and a distinctive regional atmosphere.

The interface should feel like part of a game rather than a generic SaaS dashboard.

**Visual principles:**
- Dark, near-black and charcoal foundations.
- Red-clay and rust accents.
- Warm off-white or sand-colored text.
- Restrained muted-gold details where appropriate.
- Purposeful motion and moderate corner rounding.
- Visual effects that do not compromise gameplay readability.

### DEC-004 — Existing monorepo

**Status:** Accepted

Development will continue in the existing pnpm and Turborepo monorepo.

**Implications:**
- Inspect the repository before adding applications, packages, or configuration.
- Reuse existing workspace scripts and dependencies.
- Do not recreate infrastructure that is already configured.
- Keep shared code in appropriate existing packages when practical.

### DEC-005 — Three.js for browser rendering

**Status:** Accepted

Three.js is the chosen rendering technology for the browser game.

**Implications:**
- Keep rendering code separate from unrelated application logic.
- Manage GPU resources, textures, geometries, and event listeners carefully.
- Use appropriate asset optimization and level-of-detail strategies.
- Test performance on lower-spec hardware, including integrated graphics.

### DEC-006 — Neon PostgreSQL and Drizzle ORM

**Status:** Accepted

Neon PostgreSQL is the persistent relational database, and Drizzle ORM is used for schema definitions, typed queries, and migrations.

**Implications:**
- Persistent player and game records belong in PostgreSQL.
- Database changes must follow the project's migration workflow.
- Important updates such as match rewards and currency transactions must preserve data integrity.
- Development and test operations must not accidentally modify production data.

### DEC-007 — Render hosting responsibilities

**Status:** Accepted

The intended deployment arrangement separates the live game server from static game assets:

- **Render Web Service:** Intended to host the persistent real-time game server.
- **Render Static Site:** Intended to serve public game assets such as models, textures, audio, maps, and animations.

Neon remains the persistent relational database.

**Implications:**
- Static asset delivery and real-time server hosting are separate responsibilities.
- Binary game assets should not be stored directly in PostgreSQL.
- Asset metadata and player ownership records may be stored in the database.
- The actual deployment configuration must be verified before production release.

### DEC-008 — Multiplayer is essential

**Status:** Accepted

Online multiplayer is a core requirement from the beginning, not an optional feature to add after building a single-player game.

**Implications:**
- Connection lifecycle, server authority, synchronization, and disconnections require early attention.
- The first playable milestone must include multiple connected players.
- Multiplayer testing must accompany gameplay development.

### DEC-009 — Server authority for competitive gameplay

**Status:** Accepted

The server must validate important gameplay outcomes rather than trusting client-submitted results.

This includes damage, eliminations, scores, match completion, and rewards.

**Implications:**
- Clients send inputs and requests, not authoritative final results.
- Server-side validation is required for competitive actions.
- Ranked ratings, leaderboards, and currency rewards must derive from trusted data.
- Client-side anti-cheat measures alone are insufficient.

### DEC-010 — Desktop-first combat with responsive interfaces

**Status:** Accepted

The game will support responsive non-gameplay interfaces on mobile, while competitive FPS combat remains desktop-first.

Full mobile touch-combat support is not promised for the initial release.

**Implications:**
- Landing, authentication, account, and other suitable interfaces should adapt to smaller screens.
- Desktop mouse-and-keyboard gameplay receives priority.
- Mobile layouts must not imply that complete mobile combat is already supported.

### DEC-011 — Dark-only interface

**Status:** Accepted

The game's interface will use a dark visual theme. A light-theme toggle or theme-switching system is not required.

**Implications:**
- Establish consistent dark-theme design tokens.
- Maintain readable contrast for text, controls, and gameplay overlays.
- Avoid adding unnecessary theme-switching complexity.

### DEC-012 — Lobby experience

**Status:** Accepted

The lobby should combine a cinematic 3D character and environment presentation with practical social and gameplay controls.

Players should be able to inspect their character and equipped weapon, manage customization, and access social and match-entry features.

**Implications:**
- The cinematic presentation must not obstruct essential controls.
- Character and weapon previews should reuse game assets where practical.
- Lobby rendering must remain performant.

### DEC-013 — Text chat in V1; no voice chat

**Status:** Accepted

Text chat is required for the initial version. Voice chat is excluded from V1.

**Implications:**
- Chat permissions, reporting, muting, blocking, and rate limits need appropriate support.
- The architecture should not require a voice service for the first release.
- Voice chat can be reconsidered later.

### DEC-014 — Player safety and administration

**Status:** Accepted

The game needs a way to report abusive behavior and cheating, enforce restrictions, and administer player accounts.

**Implications:**
- Important moderation actions must be authorized server-side.
- Reports and moderation actions should be recorded appropriately.
- Players should be able to mute or block others where supported.
- Sensitive reports must not expose the reporter unnecessarily.
- Administrative actions should be auditable.

### DEC-015 — Cost-conscious development

**Status:** Accepted

The project should prioritize free or low-cost development and avoid unnecessary paid infrastructure before it is justified.

**Implications:**
- Check existing services and free-tier limits before adding dependencies.
- Do not assume that a free tier can support a particular player count without testing.
- Track the cost implications of asset delivery, database traffic, and real-time hosting.
- Do not add paid services merely for convenience when a reasonable existing solution is available.

## 4. Accepted Product Targets

These are intended product targets, not claims that the features have already been implemented.

### DEC-016 — Game content targets

**Status:** Accepted

The intended content targets include:

- At least three game modes.
- At least five maps.
- Three assault rifles.
- Three submachine guns.
- Two shotguns.
- Grenades and flashbangs.
- At least five playable characters.
- A default appearance and a complimentary skin for each character.

Battle royale is intended to use larger or combined map areas where appropriate.

**Implications:**
- Separate the first playable milestone from the complete V1 content target.
- Do not silently reduce these targets without recording an explicit scope decision.
- Balance, performance, and production effort must be considered when sequencing content.

### DEC-017 — Customization and fair competition

**Status:** Accepted

Players should be able to inspect characters and weapons and equip available skins or other cosmetic items.

Cosmetics must not provide competitive advantages.

**Implications:**
- Ownership must be validated by the server.
- Equipped items must be valid for the relevant character or weapon.
- Cosmetic effects must preserve gameplay visibility and readability.

### DEC-018 — Social play

**Status:** Accepted

Social play is an important part of Red Earth. Players should be able to see online friends and invite other players to play together.

Squads and private one-versus-one play are intended social features.

**Implications:**
- Presence must distinguish online status from match availability where appropriate.
- Invitations require validation and expiration handling.
- Room membership and access must be checked by the server.
- Social features should use manageable rooms and parties rather than assuming one enormous global room.

## 5. Proposed Decisions

The following items have been recommended but must not be treated as confirmed requirements until explicitly accepted.

### PROP-001 — Build Team Deathmatch first

**Status:** Proposed

Use Team Deathmatch as the first gameplay mode for validating movement, weapons, damage, respawning, scoring, and match lifecycle.

This would establish a stable foundation before more complex modes such as battle royale.

### PROP-002 — Start with a small emote set

**Status:** Proposed

Begin with a limited selection of reusable emotes, potentially including a wave, celebration, laugh, dance, and a Red Earth-specific greeting.

Apply cooldowns and server-side rate limits. Additional emotes could become unlockable cosmetics later.

### PROP-003 — Elo-style ranked ratings

**Status:** Proposed

Use an Elo-style rating system or a suitable alternative for competitive rankings.

The final algorithm should account for match outcomes and the game's competitive structure. The system should not be finalized until the ranking requirements are agreed upon.

### PROP-004 — Progressive asset loading

**Status:** Proposed

Load essential assets first, then progressively load additional scene content and cosmetic details.

Use measurable loading progress where possible. When exact progress is unavailable, display a genuine loading or connection state rather than inventing a percentage.

### PROP-005 — Initial playable content scope

**Status:** Proposed

Prioritize one polished map and one playable mode for the first end-to-end multiplayer milestone.

This is an implementation sequence, not a replacement for the broader intended V1 content targets.

## 6. Open Decisions

### OPEN-001 — Multiplayer framework

**Status:** Open

Confirm the framework actually used by the current repository: NestJS with Socket.IO, Colyseus, or an existing combination.

The repository and installed dependencies must be inspected before declaring the framework decision final.

The decision should account for:
- Room and match lifecycle.
- Server-authoritative combat.
- State synchronization.
- Integration with the existing monorepo.
- Deployment requirements.
- Operational complexity and cost.

Do not introduce a second multiplayer framework simply because it appears in an earlier proposal.

### OPEN-002 — Final game-mode specifications

**Status:** Open

The intended minimum is three game modes, but the exact rules, player counts, scoring, respawning, and win conditions must be documented for each mode.

### OPEN-003 — Map roster and production scope

**Status:** Open

The intended minimum is five maps. The final roster, layout, landmarks, boundaries, performance budgets, and mode compatibility remain to be specified.

### OPEN-004 — Ranking algorithm

**Status:** Open

Confirm how ratings, placements, season resets, ties, and leaderboard updates should work before finalizing the ranking implementation.

### OPEN-005 — Currency and economy

**Status:** Open

Confirm the initial currency model, earning rules, spending rules, item pricing, and whether multiple currencies are necessary.

Any economy must preserve server-side authority and prevent duplicate rewards.

### OPEN-006 — Matchmaking and capacity targets

**Status:** Open

Determine target player counts per mode, room limits, matchmaking rules, and the expected number of concurrent matches.

Validate these targets with realistic load tests and hosting constraints rather than unsupported estimates.

### OPEN-007 — Authentication details

**Status:** Open

Verify the current authentication implementation and decide the supported account providers, session strategy, guest access policy, and account recovery requirements based on the existing codebase.

### OPEN-008 — Analytics and operational monitoring

**Status:** Open

Confirm the initial metrics and monitoring approach for concurrent players, completed matches, failed connections, average match duration, server health, and moderation workload.

### OPEN-009 — Font assets and licensing

**Status:** Open

Verify the availability and usage rights of the intended fonts, including Poppins and ATYPP Display, and confirm their roles in the interface.

### OPEN-010 — Voice chat after V1

**Status:** Open

Voice chat is excluded from V1. Any future inclusion requires a separate decision covering moderation, infrastructure, privacy, and cost.

## 7. Changing a Decision

When a meaningful decision changes:

1. Find the existing decision ID.
2. Record its status as **Superseded** rather than silently deleting its history.
3. Add a new decision ID describing the replacement.
4. Explain the reason for the change.
5. Update affected product, architecture, and development documents.
6. Identify any implementation or migration work required.

Small clarifications do not need a new decision ID unless they materially change scope, architecture, security, or player experience.

## 8. Rules for Coding Agents

Any coding agent working on Red Earth must:

- Read the relevant decisions before implementing a feature.
- Treat accepted decisions as constraints.
- Distinguish requirements from proposals.
- Ask for clarification when an open decision blocks implementation.
- Avoid silently changing infrastructure, frameworks, or scope.
- Inspect the repository before assuming a documented proposal has already been implemented.
- Update this document only when explicitly authorized to record a decision or when the project's agreed workflow assigns that responsibility.
- Never describe a planned feature as implemented without verifying it.

## 9. Decision Review Checklist

Before finalizing a major decision, confirm:

- [ ] The problem being solved is clear.
- [ ] Alternatives and trade-offs have been considered.
- [ ] The decision is consistent with existing accepted decisions.
- [ ] Cost and performance implications are understood.
- [ ] Security and player-safety implications are considered.
- [ ] Affected documents and implementation tasks are identified.
- [ ] The status accurately reflects the level of agreement.

---

**Guiding principle:** Accepted decisions define the direction of Red Earth. Proposals help us move forward, but they do not become requirements until they are agreed upon.