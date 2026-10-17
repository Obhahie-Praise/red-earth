# Red Earth — Development Progress

**Path:** `docs/roadmaps/PROGRESS.md`  
**Status:** Initial tracker  
**Last updated:** 2026-10-17

## 1. Purpose

This document tracks Red Earth's development from the existing monorepo to a playable, tested, and releasable browser-based multiplayer FPS.

It provides a shared view of:
- Completed work.
- Work currently in progress.
- Blockers and unresolved decisions.
- Upcoming milestones.
- Features that are planned but not yet implemented.

**Rule:** Documentation being written does not mean the corresponding feature has been implemented. Progress must be verified against the repository and, where applicable, a running application.

## 2. Status Definitions

| Status | Meaning |
|---|---|
| `DONE` | Implemented and verified against its acceptance criteria. |
| `IN PROGRESS` | Active implementation or verification is underway. |
| `BLOCKED` | Work cannot proceed until a dependency or decision is resolved. |
| `PLANNED` | Agreed work that has not started. |
| `PROPOSED` | Suggested work awaiting approval. |
| `NEEDS VERIFICATION` | There is reason to believe work exists, but its current state has not been verified. |

## 3. Current Project Snapshot

| Area | Current recorded position |
|---|---|
| Project identity | Red Earth |
| Product direction | Browser-first, online multiplayer FPS inspired by Nigeria and Edo heritage |
| Monorepo | pnpm + Turborepo; existing repository must be inspected before changes |
| Web client | Next.js, React, TypeScript, and Three.js |
| Database | Neon PostgreSQL |
| ORM | Drizzle ORM |
| Intended game-server hosting | Render Web Service |
| Intended asset delivery | Render Static Site |
| Multiplayer framework | Needs verification against the current repository |
| First playable mode | Team Deathmatch is proposed as the starting point |
| Full intended content target | At least 3 modes, 5 maps, 8 firearms, grenades, flashbangs, and 5 characters with complimentary skins |
| Mobile support | Responsive non-gameplay interfaces; desktop-first combat |
| Voice chat | Excluded from V1 |
| Cost approach | Free or low-cost first, with paid infrastructure justified by demonstrated need |

This snapshot records current direction, not proof that each system is implemented or deployed.

## 4. Development Milestones

### M0 — Repository and Development Foundation

**Goal:** Establish a reliable understanding of the existing codebase and a repeatable development workflow.

- [ ] Inspect the root workspace configuration and installed dependencies.
- [ ] Verify web and server workspace names and entry points.
- [ ] Verify the actual multiplayer framework and integration.
- [ ] Verify existing shared packages and workspace boundaries.
- [ ] Verify local development commands.
- [ ] Verify environment-variable requirements.
- [ ] Verify the Neon and Drizzle configuration.
- [ ] Confirm lint, type-check, test, and build commands.
- [ ] Confirm local web-to-server connectivity.

**Exit criteria:** A developer can install dependencies, start the required applications, connect to the development database, and run the available validation commands using documented instructions.

### M1 — Core Multiplayer Foundation

**Goal:** Establish the smallest complete online gameplay loop.

- [ ] Load the game client in a browser.
- [ ] Connect authenticated or otherwise authorized players to the server.
- [ ] Validate incoming network messages.
- [ ] Establish player movement and state synchronization.
- [ ] Spawn multiple players into the same match.
- [ ] Handle disconnections and room cleanup.
- [ ] Verify server-authoritative gameplay state.
- [ ] Test with at least two independent clients.

**Exit criteria:** Two or more players can join the same running match, observe one another, and complete the basic gameplay loop without corrupting authoritative state.

### M2 — First Playable Combat Mode

**Goal:** Deliver a complete, testable FPS match.

- [ ] Implement first-person controls and camera behavior.
- [ ] Implement the initial playable map.
- [ ] Implement the first agreed weapon set.
- [ ] Implement server-validated firing, ammunition, damage, and eliminations.
- [ ] Implement the agreed respawn rules.
- [ ] Implement scoring and match-end conditions.
- [ ] Display essential in-match information.
- [ ] Handle player departure and match cleanup.
- [ ] Test combat with multiple clients.

**Exit criteria:** Players can enter a match, fight, score points, and reach a valid match conclusion. The server validates important combat outcomes.

### M3 — Player Accounts and Persistence

**Goal:** Make player identity and persistent progress reliable.

- [ ] Verify the authentication approach.
- [ ] Implement or verify persistent player profiles.
- [ ] Connect the application to Neon PostgreSQL through Drizzle.
- [ ] Establish repeatable database migrations.
- [ ] Persist completed match results.
- [ ] Persist player statistics and progression.
- [ ] Verify inventory ownership and equipped loadouts.
- [ ] Test persistence across application and server restarts.
- [ ] Prevent duplicate match rewards.

**Exit criteria:** Relevant player and match data survives restarts and is updated through validated server-side operations.

### M4 — Lobby and Social Features

**Goal:** Give players a practical place to prepare and play together.

- [ ] Build the agreed lobby experience.
- [ ] Display the player's character and equipped weapon.
- [ ] Support character and weapon customization.
- [ ] Implement squad creation and invitations.
- [ ] Implement online-friend presence.
- [ ] Implement the agreed match-entry flow.
- [ ] Implement text chat and necessary safety controls.
- [ ] Implement private 1v1 play if confirmed in the final scope.
- [ ] Add the approved initial emote set if confirmed.

**Exit criteria:** Players can manage their loadout, interact socially, and enter matches through a coherent lobby experience.

### M5 — Content and Progression

**Goal:** Expand the first playable build toward the intended V1 content target.

- [ ] Complete the agreed minimum of 3 game modes.
- [ ] Complete the agreed minimum of 5 maps.
- [ ] Complete the target weapon roster: 3 assault rifles, 3 SMGs, and 2 shotguns.
- [ ] Implement grenades and flashbangs.
- [ ] Complete at least 5 playable characters.
- [ ] Provide a default appearance and complimentary skin for each character.
- [ ] Implement cosmetic ownership and equipment.
- [ ] Implement the agreed currency and reward rules.
- [ ] Implement progression and achievements where approved.
- [ ] Implement ranked ratings and leaderboards after the ranking rules are settled.

**Exit criteria:** The agreed V1 content is implemented, testable, and integrated with the relevant lobby, persistence, and gameplay systems.

### M6 — Safety, Performance, and Release Readiness

**Goal:** Prepare the game for a controlled public release.

- [ ] Implement player reporting and moderation workflows.
- [ ] Verify mute, block, and restriction behavior.
- [ ] Implement or verify the necessary administrative controls.
- [ ] Verify server-side permission enforcement.
- [ ] Optimize asset sizes and loading behavior.
- [ ] Test performance on lower-spec computers.
- [ ] Test network instability and reconnection behavior.
- [ ] Run multiplayer load tests and record actual results.
- [ ] Verify production database configuration and migrations.
- [ ] Verify Render server deployment and static asset delivery.
- [ ] Verify production environment variables and secrets.
- [ ] Run critical regression and end-to-end tests.
- [ ] Document known limitations and release blockers.

**Exit criteria:** Critical security, gameplay, persistence, and operational checks pass, and the release's known limitations are documented.

## 5. Active Work

No active implementation task should be listed here without evidence that work has started.

| Task | Status | Owner | Blocker / next action |
|---|---|---|---|
| Verify existing repository and workspace configuration | NEEDS VERIFICATION | TBD | Inspect the current monorepo |
| Confirm multiplayer framework | NEEDS VERIFICATION | TBD | Inspect dependencies, modules, and server entry points |
| Validate local setup instructions | NEEDS VERIFICATION | TBD | Follow `docs/development/SETUP.md` against the actual repository |
| Establish testing baseline | NEEDS VERIFICATION | TBD | Run existing test and validation scripts |

Replace these rows with verified facts as soon as the repository has been inspected. Do not leave completed setup checks marked unresolved.

## 6. Open Blockers and Decisions

Track blockers that affect implementation.

| Item | Status | Required action |
|---|---|---|
| Multiplayer framework confirmation | BLOCKED | Confirm the actual implementation before making framework-specific changes |
| Final specifications for each game mode | PLANNED | Document rules, player counts, scoring, and win conditions |
| Final map roster | PLANNED | Agree on map concepts and their gameplay requirements |
| Ranking algorithm | PLANNED | Agree on ratings, seasons, and leaderboard rules |
| Currency and economy rules | PLANNED | Define earning, spending, and ownership rules |
| Match capacity targets | PLANNED | Establish intended player counts and validate them through load tests |
| Authentication details | NEEDS VERIFICATION | Confirm current implementation and account requirements |
| Production deployment configuration | NEEDS VERIFICATION | Verify actual Render, Neon, and asset-host settings |

Update statuses as decisions are made. A question should not remain a blocker after it has been resolved.

## 7. Immediate Next Actions

Work in this order unless repository inspection reveals a more urgent issue:

1. Inspect the existing monorepo and its current implementation.
2. Correct any inaccuracies in the setup instructions.
3. Confirm the actual multiplayer framework and connection lifecycle.
4. Run the existing lint, type-check, test, and build commands.
5. Fix critical setup failures before adding new gameplay systems.
6. Identify the smallest missing pieces required for the first complete multiplayer match.
7. Begin or continue implementation according to the verified codebase.

Avoid building systems from scratch when suitable implementations already exist.

## 8. Progress Update Rules

Whenever meaningful work is completed:

1. Update the relevant milestone and task statuses.
2. Record what was implemented.
3. Record tests or manual checks that passed.
4. Note any remaining limitations.
5. Add newly discovered blockers.
6. Update relevant documentation when behavior or architecture changes.

Use specific descriptions. For example:

- **Weak:** "Multiplayer done."
- **Better:** "Two local browser clients can join one match and see server-synchronized player movement; combat validation remains incomplete."

Do not mark work `DONE` merely because code has been written. Verify the required behavior first.

## 9. Release Tracking

Before declaring a version ready:

- [ ] Required scope is agreed and documented.
- [ ] Release-critical features meet their acceptance criteria.
- [ ] Critical automated tests pass.
- [ ] A real multiplayer session has been tested.
- [ ] Persistent data has been verified.
- [ ] Production configuration has been checked.
- [ ] Known critical bugs are resolved.
- [ ] Remaining limitations are documented.
- [ ] Deployment and rollback procedures are understood.

Record the release version, verification date, and known limitations when a release is actually prepared.

## 10. Relationship to Other Documents

- `docs/product/VISION.md` — product direction and principles.
- `docs/product/GAME_DESIGN.md` — gameplay and player experience.
- `docs/product/FEATURES.md` — feature requirements and acceptance criteria.
- `docs/architecture/OVERVIEW.md` — system architecture.
- `docs/architecture/MONOREPO.md` — workspace structure and conventions.
- `docs/architecture/MULTIPLAYER.md` — multiplayer design.
- `docs/architecture/SECURITY.md` — security and player safety.
- `docs/architecture/REALTIME-PROTOCOL.md` — real-time message design.
- `docs/architecture/DATA-MODEL.md` — persistence and data model.
- `docs/development/CODE_STANDARDS.md` — coding conventions.
- `docs/development/TESTING.md` — testing strategy.
- `docs/development/SETUP.md` — local development setup.
- `docs/roadmaps/DECISIONS.md` — accepted decisions, proposals, and open questions.

**Guiding principle:** This tracker must reflect the real state of the codebase, not the state we hope it is in.