# Red Earth — Development Roadmap

**Path:** `docs/roadmaps/ROADMAP.md`  
**Status:** Initial roadmap  
**Last updated:** 2026-10-17

## 1. Overview

Red Earth is a browser-first multiplayer FPS inspired by Nigeria, with a distinctive Edo and Benin City identity. The goal is to build a responsive, visually memorable online game that players can access through their browsers without installing a standalone game.

This roadmap defines the intended development sequence from the existing monorepo to a playable release.

It is a planning document, not a claim that the listed work has not started. Before scheduling implementation, inspect the current repository and reconcile this roadmap with `PROGRESS.md`.

## 2. Roadmap Principles

- **Build on the existing codebase.** Inspect before creating or replacing anything.
- **Prove multiplayer early.** A real match with multiple connected players is more valuable than a large collection of disconnected features.
- **Keep the server authoritative.** Important combat outcomes, match results, and rewards must be validated by the server.
- **Develop toward the full content target.** The first playable milestone is not the final V1 scope.
- **Prioritize performance.** The game should remain practical for players using lower-spec computers and integrated graphics.
- **Protect the project's budget.** Avoid unnecessary paid infrastructure and services until their value is demonstrated.
- **Validate as we build.** Testing, security, and performance work should accompany implementation.
- **Keep decisions explicit.** Follow `docs/roadmaps/DECISIONS.md` and resolve blockers before making assumptions.

## 3. Roadmap at a Glance

<text color="secondary" size="sm">Phases are ordered by dependency, not assigned calendar dates. Dates should be added after the existing implementation and available development capacity have been assessed.</text>

<box border radius="lg" padding={3} gap={3}>
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="md" size="40px" align="center" justify="center">
      <icon name="wrench" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Phase 0 — Verify the foundation**

      <text color="secondary" size="sm">Existing monorepo, local setup, database, server framework, scripts, and environment.</text>
    </box>
  </row>
  <divider color="subtle" />
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="md" size="40px" align="center" justify="center">
      <icon name="users" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Phase 1 — Multiplayer foundation**

      <text color="secondary" size="sm">Connections, rooms, player synchronization, lifecycle, and two-client testing.</text>
    </box>
  </row>
  <divider color="subtle" />
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="md" size="40px" align="center" justify="center">
      <icon name="crosshair" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Phase 2 — First complete FPS match**

      <text color="secondary" size="sm">Movement, camera, map, weapons, combat, scoring, respawns, and match completion.</text>
    </box>
  </row>
  <divider color="subtle" />
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="md" size="40px" align="center" justify="center">
      <icon name="database" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Phase 3 — Accounts and persistence**

      <text color="secondary" size="sm">Player profiles, Neon, Drizzle, match history, progression, and inventory.</text>
    </box>
  </row>
  <divider color="subtle" />
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="md" size="40px" align="center" justify="center">
      <icon name="gamepad-2" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Phase 4 — Lobby and social play**

      <text color="secondary" size="sm">Character previews, loadouts, parties, invitations, friends, and text chat.</text>
    </box>
  </row>
  <divider color="subtle" />
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="md" size="40px" align="center" justify="center">
      <icon name="map" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Phase 5 — Full V1 content and progression**

      <text color="secondary" size="sm">Game modes, maps, weapon roster, characters, cosmetics, and agreed progression systems.</text>
    </box>
  </row>
  <divider color="subtle" />
  <row align="start" gap={3}>
    <box background="surface-secondary" radius="md" size="40px" align="center" justify="center">
      <icon name="shield-check" size="xl" />
    </box>
    <box flex="1" gap={1}>
      **Phase 6 — Hardening and release**

      <text color="secondary" size="sm">Moderation, anti-cheat validation, optimization, load testing, deployment, and release checks.</text>
    </box>
  </row>
</box>

## 4. Phase 0 — Verify the Existing Foundation

**Objective:** Understand what already exists and make the development environment reliable before expanding the game.

### Work

- Inspect the pnpm/Turborepo workspace structure.
- Verify the web and server applications.
- Confirm the actual multiplayer framework and integration.
- Verify shared packages and dependency boundaries.
- Confirm Neon and Drizzle configuration.
- Verify the intended Render deployment arrangement.
- Confirm local asset-loading conventions.
- Run available lint, type-check, test, and build commands.
- Correct documentation that does not match the codebase.

### Exit criteria

A developer can run the existing applications locally, verify the relevant database connection, and understand how the client communicates with the server.

**Dependency:** None. This phase begins with the repository as it currently exists.

## 5. Phase 1 — Multiplayer Foundation

**Objective:** Establish a reliable real-time connection between multiple browser clients.

### Work

- Finalize the multiplayer framework decision based on the existing repository.
- Implement or verify connection handling.
- Establish player identity and session validation.
- Implement room or match membership.
- Synchronize player movement and essential state.
- Handle joining, leaving, disconnects, and cleanup.
- Validate incoming messages.
- Prevent players from joining unauthorized matches.
- Establish basic network diagnostics.

### Exit criteria

At least two independent clients can join the same session, see synchronized player state, and disconnect without corrupting the remaining session.

### Important constraint

Do not build elaborate matchmaking, ranking, or social infrastructure before the basic multiplayer lifecycle works reliably.

## 6. Phase 2 — First Complete FPS Match

**Objective:** Turn the multiplayer foundation into a playable FPS.

### Work

- Implement or complete first-person controls and camera behavior.
- Create the first playable map using Red Earth's visual direction.
- Implement the initial weapon handling and ammunition rules.
- Implement firing, hit validation, damage, and eliminations.
- Add respawning and scoring.
- Implement match start, completion, and cleanup.
- Display essential in-match information.
- Add basic audio and combat feedback where appropriate.
- Test gameplay with multiple clients under different network conditions.

### Exit criteria

Players can join a match, move, fight, eliminate opponents, score, respawn according to the agreed rules, and finish a match.

The server validates important gameplay outcomes.

### Suggested first mode

Team Deathmatch is the proposed starting mode because it allows movement, combat, respawning, scoring, and match lifecycle to be tested without requiring battle-royale-scale systems.

This recommendation does not remove the broader three-mode target.

## 7. Phase 3 — Accounts and Persistent Progress

**Objective:** Ensure players retain their identity, results, and earned items.

### Work

- Verify the authentication and session implementation.
- Implement or complete persistent player profiles.
- Connect application modules to Neon through Drizzle.
- Establish reliable schema migrations.
- Persist completed match results and player statistics.
- Implement inventory ownership and loadout persistence.
- Implement progression and rewards according to approved rules.
- Prevent duplicate result processing and duplicate rewards.
- Test database behavior across server restarts.

### Exit criteria

Player records, completed match history, and approved progression data persist reliably. Important changes are validated and cannot be duplicated through repeated requests.

## 8. Phase 4 — Lobby and Social Play

**Objective:** Give players a coherent place to customize their characters and organize matches.

### Work

- Implement the lobby layout and cinematic character presentation.
- Display equipped characters, weapons, and cosmetics.
- Add character and weapon inspection.
- Implement valid loadout changes.
- Add squad creation and invitations.
- Display friend presence.
- Implement the agreed match-entry flow.
- Implement text chat with appropriate controls and moderation.
- Add private 1v1 play if retained in the final feature scope.
- Add the initial emote set if approved.

### Exit criteria

Players can enter the lobby, manage their loadouts, interact with other players, and organize gameplay without bypassing server-side permissions.

### Design constraint

The lobby should feel like a game environment, not an administrative dashboard. Its 3D presentation must remain performant and must not obstruct essential controls.

## 9. Phase 5 — Full V1 Content and Progression

**Objective:** Expand the first complete playable match into the intended Red Earth V1 experience.

### Content targets

| Category | Intended target |
|---|---|
| Game modes | At least 3 |
| Maps | At least 5 |
| Assault rifles | 3 |
| Submachine guns | 3 |
| Shotguns | 2 |
| Explosives | Grenades and flashbangs |
| Playable characters | At least 5 |
| Character cosmetics | Default appearance and a complimentary skin for each character |

Battle royale should use appropriately sized or combined map areas, with the exact implementation determined by map design and performance requirements.

### Work

- Expand the weapon roster.
- Implement the remaining agreed game modes.
- Produce the remaining maps and their assets.
- Complete the character roster and complimentary skins.
- Implement cosmetic inventory and equipment.
- Add the approved currency and reward system.
- Add progression and achievements where agreed.
- Finalize ranked ratings and leaderboards after the rules are approved.
- Verify that content works across the relevant maps and modes.

### Exit criteria

The agreed V1 content targets are implemented, tested, and integrated with multiplayer, persistence, customization, and the lobby.

### Scope protection

Do not confuse the first playable milestone with the complete V1 release. Conversely, do not silently reduce the intended content targets to meet a milestone. Any scope reduction requires an explicit decision recorded in `DECISIONS.md`.

## 10. Phase 6 — Security, Performance, and Release

**Objective:** Prepare Red Earth for a controlled release.

### Security and moderation

- Verify server-side authorization for gameplay and administrative actions.
- Validate and rate-limit network messages.
- Test chat restrictions, muting, blocking, and reporting.
- Implement or complete moderation workflows.
- Protect secrets and sensitive player data.
- Record administrative actions appropriately.

### Performance and reliability

- Optimize models, textures, audio, and map assets.
- Implement appropriate asset-loading and caching strategies.
- Reduce unnecessary draw calls and expensive scene operations.
- Clean up rendering resources correctly.
- Test on lower-spec computers.
- Measure server capacity under representative workloads.
- Test latency, jitter, packet loss, and disconnections.
- Verify database query performance and failure handling.

### Deployment

- Verify the production game-server configuration.
- Verify Neon production connectivity and migrations.
- Verify Render Static Site asset delivery.
- Configure production environment variables and secrets.
- Verify client-to-server connectivity in the deployed environment.
- Establish logging, operational metrics, and recovery procedures.

### Exit criteria

Critical gameplay, security, persistence, and operational checks pass. The team understands the game's measured performance limits and has documented known issues.

## 11. Work That Can Run in Parallel

Some work can proceed alongside gameplay development, provided it does not depend on unresolved architectural choices.

| Workstream | Can begin when |
|---|---|
| Visual identity and UI refinement | Existing app structure and design direction are understood |
| Character and weapon concept development | Asset requirements and visual direction are documented |
| Map concepts and reference gathering | Gameplay requirements for the maps are understood |
| Database schema and migrations | Relevant persistent-data requirements are agreed |
| Automated tests | The behavior being tested has a clear contract |
| Asset optimization workflow | The current asset pipeline has been inspected |
| Moderation design | Reporting requirements and authorization boundaries are defined |

Parallel work should not create incompatible assumptions about APIs, shared types, asset formats, or gameplay rules.

## 12. Deferred Beyond V1

The following are not part of the initial committed scope unless explicitly brought forward through a recorded decision:

- Voice chat.
- Full mobile touch-combat support.
- Complex economic systems beyond the approved initial currency model.
- Advanced competitive systems beyond the agreed ranking requirements.
- Additional large-scale social systems beyond the initial lobby and squad experience.
- Expensive infrastructure introduced before demonstrated need.

Deferring a feature does not mean it can never be added. It means the initial release should not depend on it.

## 13. Milestone Planning and Prioritization

Do not assign arbitrary delivery dates before the existing implementation has been assessed.

For each milestone:

1. Review `PROGRESS.md` to establish the actual starting point.
2. Identify dependencies and unresolved decisions.
3. Break work into small, verifiable tasks.
4. Estimate effort based on the real implementation.
5. Prioritize blockers and release-critical behavior.
6. Test each completed feature against its acceptance criteria.
7. Update `PROGRESS.md` and relevant documentation.
8. Reassess the next milestone based on actual results.

When choosing between new content and fixing a broken foundation, prioritize the foundation if the defect blocks core gameplay, security, or reliable testing.

## 14. Roadmap Maintenance

Update this document when:
- A major milestone is completed or materially restructured.
- An accepted decision changes the implementation order.
- A significant dependency or risk is discovered.
- The V1 scope changes through an explicit decision.
- A previously deferred feature becomes a committed priority.

Use `PROGRESS.md` for detailed implementation status and `DECISIONS.md` for the reasoning behind major changes.

**Guiding principle:** Build a complete, reliable multiplayer experience first, then expand it into the full Red Earth world without sacrificing its identity, performance, or competitive integrity.