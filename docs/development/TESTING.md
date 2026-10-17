# Testing Strategy

**Project:** Red Earth  
**Path:** `docs/development/TESTING.md`  
**Status:** Draft

## 1. Purpose

This document defines how Red Earth is tested to catch bugs early, protect multiplayer integrity, and keep the game stable as new features are added.

Testing should be practical for a small development team. Prioritize tests that protect gameplay, player data, networking, and security rather than chasing an arbitrary test-coverage percentage.

## 2. Testing Principles

- **Test important behavior, not implementation details.**
- **Protect multiplayer integrity.** The server must never trust a client's claims about damage, kills, scores, or match results.
- **Test shared contracts.** Client and server must agree on message shapes and validation rules.
- **Keep tests repeatable.** Tests should not depend on real players, unstable internet connections, or paid external services.
- **Test real integrations where necessary.** Mocks alone cannot prove that the database or networking layer works correctly.
- **Test performance early.** A browser game that runs poorly on the target hardware is not ready.
- **Fix regressions.** When a bug is discovered, add a test when practical so the same problem is less likely to return.

## 3. Testing Levels

### 3.1 Unit tests

Test individual functions, classes, and modules in isolation.

Examples:
- Weapon damage and fire-rate calculations.
- Ammunition and reload rules.
- Movement and gameplay validation.
- Score, XP, and progression calculations.
- Rank changes and leaderboard calculations.
- Currency transactions and ownership checks.
- Input validation and permission checks.
- Shared message-schema validation.
- UI state transitions and formatting.

Use the repository's existing **Vitest** setup where appropriate. Do not introduce another test runner without a clear reason.

### 3.2 Integration tests

Test whether connected parts of the system work together correctly.

Examples:
- NestJS modules interacting with the database layer.
- Drizzle queries against a test Neon PostgreSQL database or an isolated PostgreSQL test instance compatible with the production schema.
- Database migrations applying successfully.
- Authentication and session handling.
- Socket connections, room membership, and disconnections.
- Match results being persisted correctly.
- Inventory ownership and loadout changes.
- Reporting, blocking, and moderation actions.
- Shared contracts being enforced at network boundaries.

Use isolated test data and clean up after each test. Never run destructive test setup against production data.

### 3.3 End-to-end tests

Test complete player journeys through the application.

Priority journeys include:

1. A player opens the game and reaches the main interface.
2. A player signs in and loads their profile.
3. A player enters the lobby and sees their equipped character and weapon.
4. Players form a squad or invite another player.
5. A player joins a match and completes a match cycle.
6. Match results update the correct statistics and progression.
7. A player changes a loadout and sees the change persist.
8. A player reports or blocks another player.
9. A disconnected player can reconnect or return to a valid state.

Automate stable browser journeys where practical. Keep multiplayer scenarios separate from basic UI tests so failures are easier to diagnose.

## 4. Web Client Testing

The web client includes the application interface, Three.js rendering, input handling, and real-time networking.

Test the following:

- Main navigation and page transitions.
- Authentication, profile loading, and error states.
- Lobby, squad, customization, and loadout interfaces.
- Keyboard and mouse input.
- Camera behavior and movement integration.
- Connection status, disconnects, and reconnecting.
- Loading states for maps and assets.
- Missing or failed asset handling.
- Responsive layouts on supported mobile screen sizes.
- Cleanup of event listeners, network connections, and Three.js resources.
- Graceful handling of server errors and invalid responses.

**Rendering checks:** Verify that scenes load, essential objects appear, controls work, and the game can recover from asset-loading failures. Do not rely exclusively on screenshots; visual snapshots cannot prove that gameplay works.

## 5. Multiplayer and Gameplay Testing

Multiplayer correctness is a release-critical area.

### 5.1 Server authority

Verify that the server independently validates:

- Player identity and match membership.
- Movement limits and invalid position updates.
- Weapon selection and ammunition.
- Fire rate, reload timing, and weapon restrictions.
- Hit claims, damage, eliminations, and respawns.
- Team membership and friendly-fire rules, where applicable.
- Scores, match completion, and rewards.
- Ranked-rating changes and leaderboard updates.

A modified client must not be able to award itself kills, change its own health, invent currency, or submit a trusted match result.

### 5.2 Room and session lifecycle

Test:

- Joining and leaving rooms.
- Duplicate join requests.
- Unexpected disconnects.
- Reconnection and expired sessions.
- Players leaving during active matches.
- Empty-room cleanup.
- Room capacity limits.
- Invalid messages and unauthorized actions.
- Multiple matches operating without state leaking between them.

### 5.3 Multi-client scenarios

Use multiple independent clients or test clients to verify that:

- Players see consistent match state.
- Movement and combat events reach the correct participants.
- One player's disconnection does not crash another player's match.
- Team and squad boundaries are respected.
- Late or repeated messages do not corrupt state.
- Simultaneous actions do not duplicate rewards or results.

Test with at least two clients for core multiplayer flows, then increase the number as performance and matchmaking develop.

## 6. Database and Data Integrity

Using Neon PostgreSQL and Drizzle ORM, test that:

- The schema and migrations can build a fresh database.
- Existing migrations can upgrade a database to the latest schema.
- Player profiles and match histories persist across restarts.
- Inventory cannot contain unauthorized ownership claims.
- Currency changes are validated and recorded consistently.
- Duplicate match-result submissions cannot grant duplicate rewards.
- Ranked ratings and leaderboard data use trusted server results.
- Deleting or restricting an account does not break related records.
- Database errors are handled without exposing secrets or sensitive details.

Use transactions for operations that must succeed or fail together. Test concurrency-sensitive operations, especially purchases, rewards, and match-result processing.

Never depend on manually editing production data to make a test pass.

## 7. Security and Abuse Testing

At minimum, test the following:

- Unauthenticated access to protected actions.
- Attempts to perform admin actions without the correct role.
- Malformed, oversized, and unexpected network messages.
- Excessive message rates and repeated requests.
- Attempts to join unauthorized rooms or matches.
- Chat spam, blocked users, and restricted accounts.
- Duplicate rewards and replayed match submissions.
- Injection attempts in user-controlled text and profile fields.
- Secret exposure in logs, client bundles, and error responses.

Tests must confirm that important authorization checks happen on the server. Hiding a button in the interface is not sufficient protection.

## 8. Performance and Reliability Testing

Red Earth must be tested on hardware representative of its intended audience, including lower-spec computers and integrated graphics.

### Client performance

Measure:
- Frame rate during ordinary gameplay and combat.
- Initial loading time and time to playable state.
- Memory use during longer sessions.
- Asset download sizes and scene-loading behavior.
- Performance when multiple characters and effects are visible.
- Resource cleanup when changing maps or leaving matches.

### Server performance

Measure:
- Concurrent connected players.
- Active matches and room counts.
- Network message frequency and payload size.
- Server CPU and memory usage.
- Snapshot or state-update processing time.
- Database query latency.
- Disconnect rates and failed match joins.

### Network conditions

Simulate or manually test:
- High latency.
- Jitter and packet loss.
- Temporary disconnections.
- Slow asset downloads.
- Server restarts and failed connections.

Record actual results. Do not claim a supported player count until representative load tests demonstrate it.

## 9. Test Data and Environments

Keep development, testing, and production data separate.

- Use synthetic accounts and predictable fixtures.
- Use isolated database instances or dedicated test databases.
- Avoid real player credentials and personal information in fixtures.
- Make tests repeatable without manually preparing database records.
- Clean up temporary rooms, accounts, and records created by tests.
- Keep production secrets out of local test files and CI logs.
- Mock external dependencies when testing their consumers in isolation; use real test integrations when validating the integration itself.

Do not make the test suite depend on a paid service or a live external provider unless the test specifically verifies that integration.

## 10. Continuous Integration and Local Validation

The repository's existing pnpm and Turborepo setup should coordinate validation across affected packages.

The normal validation sequence should include:

1. Formatting or formatting checks, according to the repository's configured tooling.
2. ESLint or the configured lint command.
3. TypeScript type-checking.
4. Unit tests.
5. Relevant integration tests.
6. Production builds for affected applications and packages.

Run focused tests while developing, then run the broader checks before merging or releasing.

Use the actual scripts defined in the root `package.json` and individual workspace manifests. Do not assume a command exists before verifying it. Keep the CI pipeline aligned with local commands to avoid different behavior between environments.

Longer-running browser, multiplayer, and load tests may run in a separate CI stage when running them on every change would be unnecessarily expensive.

## 11. Test Priorities

| Priority | What must be tested |
|---|---|
| Critical | Authentication, server authority, combat validation, match results, permissions, data integrity |
| High | Multiplayer lifecycle, reconnects, database persistence, inventory, squads, chat safety |
| Normal | UI behavior, customization, loading failures, responsive layouts, progression |
| Ongoing | Large-scale load, visual polish, advanced matchmaking, broader device coverage |

Critical tests should be automated before the corresponding feature is considered ready for release.

## 12. Definition of Done

A feature is not considered complete until:

- Its important success and failure cases have been tested.
- Relevant automated tests pass.
- Type-checking and lint checks pass.
- Network inputs and permissions are validated where applicable.
- Persistent changes behave correctly across reloads or server restarts.
- Loading, disconnect, and error states have been considered.
- Relevant documentation has been updated.
- No known critical regression remains unresolved.

For gameplay features, also verify the behavior with actual clients in a running match.

## 13. Bug Regression Workflow

When a bug is found:

1. Record the steps that reproduce it.
2. Identify the affected client, server, shared package, or database layer.
3. Add a regression test when practical.
4. Fix the underlying cause rather than only hiding the symptom.
5. Run the focused test and relevant broader checks.
6. Update the issue or progress tracker with the result.

For multiplayer bugs, record useful diagnostic information such as match identifiers, event types, timestamps, and server-side errors. Never log passwords, session tokens, or other secrets.

## 14. Implementation Order

Adopt testing alongside feature development rather than postponing it until the game is complete.

1. Establish reliable unit-test and type-check commands across the existing workspaces.
2. Test shared contracts and network-message validation.
3. Add unit tests for authoritative gameplay rules.
4. Add database integration tests for persistence and rewards.
5. Add two-client tests for the first playable multiplayer mode.
6. Add browser tests for the critical player journey.
7. Add performance baselines and progressively increase load.
8. Expand regression coverage as features and real-world usage expose new failure cases.

**Guiding rule:** Every release-critical system needs a repeatable way to verify that it works. Red Earth does not need exhaustive tests for everything on day one, but it must not rely on hope for combat integrity, multiplayer stability, or player data.