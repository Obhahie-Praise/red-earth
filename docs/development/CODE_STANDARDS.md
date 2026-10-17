# Red Earth — Code Standards

**Path:** `docs/development/CODE_STANDARDS.md`  
**Status:** Draft  
**Applies to:** All applications and packages in the Red Earth monorepo.

## 1. Purpose

These standards keep Red Earth maintainable as the game grows across its browser client, multiplayer server, and shared packages.

The goal is consistent, readable code without unnecessary abstractions or rules that slow development.

All contributors and coding agents must follow these standards. When existing repository conventions differ from this document, inspect the code first and identify the discrepancy rather than blindly rewriting established code.

## 2. Core Principles

1. Prefer simple, readable implementations over clever abstractions.
2. Use strict TypeScript and avoid `any`.
3. Keep responsibilities separated between the client, server, and shared packages.
4. Validate all untrusted data at system boundaries.
5. Keep gameplay rules testable and independent of rendering where practical.
6. Avoid duplicate logic for important game rules.
7. Follow existing project conventions unless there is a clear reason to change them.
8. Do not introduce dependencies or infrastructure without a demonstrated need.
9. Preserve established UI and visual design unless the task explicitly requires changes.
10. Make focused changes instead of rewriting unrelated files.

## 3. TypeScript

Use TypeScript for application and shared-package code.

### Required rules

- Keep strict compiler options enabled wherever supported by the existing configuration.
- Do not use `any`. Prefer precise types, generics, discriminated unions, or `unknown` with validation.
- Avoid unsafe type assertions. When an assertion is necessary, explain why the value is safe.
- Use explicit return types for exported functions and public APIs where they improve clarity.
- Avoid non-null assertions unless the invariant is guaranteed and documented.
- Use `unknown` rather than `any` for untrusted values.
- Narrow types before accessing optional or union-specific properties.
- Avoid duplicating types that can be imported from an appropriate shared package.
- Do not use TypeScript types as a replacement for runtime validation.

Example:

```ts
// Avoid
function handleMessage(message: any) {
  return message.playerId;
}

// Prefer
interface PlayerMessage {
  playerId: string;
}

function handleMessage(message: PlayerMessage): string {
  return message.playerId;
}
```

For untrusted network input, the second example alone is not enough. Validate the payload at runtime before treating it as a `PlayerMessage`.

## 4. Naming Conventions

Use consistent, descriptive names.

| Element | Convention | Example |
|---|---|---|
| Variables | camelCase | `currentMatch` |
| Functions | camelCase | `calculateRating` |
| Types and interfaces | PascalCase | `PlayerProfile` |
| Classes | PascalCase | `MatchService` |
| React components | PascalCase | `PlayerCard` |
| Constants | camelCase by default | `defaultLoadout` |
| Environment variables | UPPER_SNAKE_CASE | `DATABASE_URL` |
| Database tables | Follow established Drizzle schema conventions | `playerProfiles` or `player_profiles` |
| Files | Follow the containing framework's conventions | `match.service.ts` |

Do not rename existing files or change naming conventions across the repository simply to enforce personal preference.

Choose names that communicate purpose. Avoid vague names such as `data`, `stuff`, `helper`, and `manager` when a more specific name is available.

## 5. Project Structure and Responsibilities

Keep code in the correct application or package.

### Browser application

The web app owns:

- Rendering and interface interactions.
- Three.js scene setup and presentation.
- Keyboard, mouse, and supported device input.
- Client-side visual effects and audio playback.
- UI state and non-authoritative prediction.
- Network communication with the server.

The browser must not be the authority for competitive damage, scores, inventory ownership, currency, or ranked results.

### Multiplayer server

The server owns:

- Authentication and authorization checks.
- Match and room lifecycle.
- Authoritative movement and combat rules.
- Weapon validation and damage.
- Match results and scoring.
- Rewards and progression processing.
- Server-side rate limits and abuse protection.

Keep game rules separate from transport-specific handlers when doing so makes them easier to test and maintain.

### Shared packages

Shared packages may contain:

- Network contracts.
- Shared game types.
- Runtime validation schemas.
- Stable identifiers and enums.
- Pure utilities used by multiple applications.

Do not put browser-only rendering code or server-only secrets into shared packages.

Do not create a shared package merely to hold a handful of unrelated helpers.

## 6. React and Frontend Standards

Use the React and Next.js patterns already established by the project.

- Prefer functional components and hooks.
- Keep components focused on a clear responsibility.
- Extract reusable logic when duplication or complexity justifies it.
- Avoid unnecessary global state.
- Keep server-only code out of client components.
- Use the framework's established routing and data-fetching patterns.
- Handle loading, error, empty, and success states.
- Avoid unnecessary effects when a value can be derived directly.
- Clean up subscriptions, event listeners, and resources when their lifecycle ends.

Do not move everything into a generic component library. Reuse components where reuse genuinely improves consistency.

### UI preservation

Red Earth's established visual direction should remain consistent:

- Dark-only interface.
- Near-black and charcoal surfaces.
- Red-clay and rust accents.
- Warm off-white or sand-colored text.
- Restrained muted-gold accents.
- Poppins and the agreed display typography, using the actual licensed font files available in the repository.
- Moderate rounding and purposeful animation.
- Game-native interface design rather than a generic SaaS dashboard.

Do not introduce a theme switcher or unrelated design system without explicit approval.

When implementing a feature, inspect the relevant screens and existing components before modifying them.

## 7. Three.js and Game Client Standards

The game client must account for browser performance and resource management.

- Reuse geometries, materials, and textures where appropriate.
- Avoid creating expensive objects repeatedly inside the render loop.
- Dispose of GPU resources when they are no longer needed.
- Remove event listeners and animation loops during cleanup.
- Avoid unnecessary allocations in frequently executed update functions.
- Keep rendering and authoritative gameplay logic separate.
- Load assets progressively when doing so improves startup.
- Use level of detail, compressed assets, and appropriate texture sizes where justified.
- Avoid expensive scene-wide operations on every frame.
- Profile performance before attempting speculative optimizations.

Do not use React state updates for every high-frequency simulation change when a more suitable update mechanism is available.

Keep rendering smooth without allowing the client to determine authoritative gameplay outcomes.

## 8. NestJS and Multiplayer Server Standards

Follow the existing NestJS module and dependency-injection conventions.

- Keep controllers and transport handlers thin.
- Put application logic in appropriate services.
- Separate domain rules from transport concerns where practical.
- Validate incoming HTTP and real-time payloads.
- Check authorization on the server.
- Avoid circular module dependencies.
- Use explicit interfaces and types for service boundaries.
- Clean up connections, listeners, and temporary state appropriately.
- Handle expected failures without crashing the server.
- Log unexpected failures with useful context, without exposing secrets.

### Multiplayer-specific rules

- Never trust client-supplied damage or final scores.
- Never trust a client-supplied account role.
- Validate weapon state, movement, and match transitions.
- Use server-owned state for combat and rewards.
- Make important persistent operations idempotent.
- Keep transient match state separate from durable PostgreSQL records.

The implementation must follow the selected multiplayer framework's conventions. Do not mix incompatible Colyseus and Socket.IO patterns without an explicit architecture decision.

## 9. Database and Drizzle Standards

Neon PostgreSQL is the persistent database, and Drizzle ORM manages the schema and database access.

- Keep schema definitions in the repository's established location.
- Use migrations for schema changes.
- Review generated migrations before applying them.
- Avoid production schema changes that bypass the migration history.
- Use transactions for operations that must succeed or fail together.
- Add constraints to protect important data relationships.
- Use parameterized, ORM-safe queries.
- Avoid unbounded queries and unnecessary database round trips.
- Add indexes based on actual query patterns.
- Never expose database credentials to the browser.

Currency, inventory ownership, match results, ranked ratings, and rewards require special care around transactions, concurrent requests, and duplicate processing.

Do not persist every simulation tick or player position to PostgreSQL.

## 10. Error Handling

Handle expected errors explicitly.

Examples include:

- Invalid user input.
- Expired sessions.
- Missing resources.
- Unauthorized actions.
- Full rooms.
- Invalid match states.
- Rate limits.
- Temporary database failures.

Use consistent error structures at application boundaries.

Do not silently swallow exceptions or return fake successful results when an operation fails.

Avoid exposing internal stack traces, database details, secrets, or sensitive security information to users.

When an operation has partially completed, use transactions or a documented recovery strategy rather than pretending everything succeeded.

## 11. Logging

Logs should explain what happened and help diagnose problems.

- Use the project's established logging mechanism.
- Include relevant identifiers, such as match IDs or request IDs, where appropriate.
- Log unexpected failures with enough context to investigate them.
- Avoid logging passwords, session tokens, API keys, and unnecessary personal data.
- Do not log every frame or every movement update.
- Keep high-frequency gameplay telemetry bounded and intentional.

Prefer structured logs when supported by the existing stack.

## 12. Testing Standards

Write tests for behavior that matters, particularly authoritative gameplay and persistent state changes.

### Unit tests

Use unit tests for:

- Weapon and combat rules.
- Movement validation.
- Match-state transitions.
- Ranking calculations.
- Reward eligibility.
- Currency operations.
- Input validation.
- Authorization decisions.

### Integration tests

Test:

- Database transactions and migrations.
- Authentication and protected operations.
- Room joining and leaving.
- Real-time message handling.
- Match-result persistence.
- Inventory and equipment validation.
- Duplicate reward prevention.
- Moderation restrictions.

### Frontend tests

Test important user interactions and states, especially loading, error handling, equipment selection, and network failures.

### General rules

- Use the test framework already installed in the relevant package.
- Make tests deterministic where possible.
- Avoid tests that depend on production services or real user accounts.
- Do not weaken assertions simply to make a failing test pass.
- Fix the underlying issue when a test reveals a real defect.

A successful build is not proof that multiplayer behavior or security rules work correctly.

## 13. Dependencies and Package Management

Red Earth uses pnpm and Turborepo.

- Use pnpm for package installation and workspace commands.
- Keep dependencies in the package that actually uses them.
- Avoid adding a dependency when a suitable existing solution is available.
- Prefer maintained packages with a clear purpose.
- Review runtime, browser, and server compatibility before adding dependencies.
- Avoid unnecessary overlapping libraries.
- Do not introduce paid services or infrastructure without approval.
- Avoid adding Docker or other local-development requirements unless the project needs them.

Use the existing workspace configuration and package scripts. Do not recreate the monorepo or replace established tooling to implement an ordinary feature.

## 14. Environment Variables and Secrets

- Keep local environment files out of Git.
- Use the repository's existing environment-variable conventions.
- Validate required environment variables at application startup.
- Document variable purpose and whether it is public or secret.
- Keep server secrets in server-only environments.
- Do not expose secrets through `NEXT_PUBLIC_` variables or client bundles.
- Use placeholders in examples and documentation.
- Never overwrite production credentials or secrets without explicit authorization.

If a required environment variable is missing, fail clearly rather than silently substituting a misleading default.

## 15. Performance Standards

Optimize based on measurement.

For the browser:

- Monitor frame time, memory, asset loading, and network traffic.
- Avoid unnecessary re-renders and per-frame allocations.
- Manage Three.js resources correctly.
- Consider lower-spec devices when choosing asset quality.

For the server:

- Monitor simulation duration, connection count, message rates, and memory usage.
- Avoid blocking operations in latency-sensitive paths.
- Keep network payloads compact.
- Avoid database writes in high-frequency simulation loops.
- Measure room capacity before claiming a supported player count.

For PostgreSQL:

- Inspect slow queries.
- Use suitable indexes.
- Avoid N+1 query patterns.
- Paginate large result sets.
- Use transactions for critical updates.

Do not sacrifice correctness or competitive integrity for a speculative performance improvement.

## 16. Accessibility and Responsive Interfaces

The game is desktop-first for FPS combat, but non-gameplay interfaces should support responsive layouts.

- Keep navigation and account controls usable on smaller screens.
- Provide clear focus states for keyboard navigation.
- Respect reduced-motion preferences where practical.
- Maintain readable contrast.
- Use descriptive labels for interactive controls.
- Do not communicate important status through color alone.
- Handle loading and error states clearly.

Full mobile touch-based FPS combat is not assumed to be part of V1.

## 17. Documentation and Comments

Write comments to explain why a non-obvious decision exists, not to restate what the code visibly does.

Document:

- Important architectural decisions.
- Non-obvious game rules.
- Security boundaries.
- Data consistency requirements.
- Workarounds and known limitations.
- Public contracts and configuration.

Update relevant documentation when a code change alters architecture, commands, configuration, or expected behavior.

Do not create extra documentation files outside the agreed plan merely because a topic could have its own document.

## 18. Git and Change Management

- Make focused changes related to the task.
- Do not overwrite unrelated user work.
- Do not remove existing features without authorization.
- Do not commit secrets or generated build artifacts.
- Inspect `git diff` before completing a change.
- Report any unrelated issues discovered instead of silently fixing everything.
- Do not claim tests passed unless they were actually run.
- Do not claim a feature is complete if only its UI or placeholder exists.

Do not perform destructive Git operations or rewrite shared history without explicit approval.

## 19. Required Workflow for Coding Agents

Before implementing a feature:

1. Read the relevant product and architecture documents.
2. Inspect the existing repository and affected files.
3. Confirm the current package scripts and dependencies.
4. Identify the smallest coherent implementation.
5. Follow established patterns unless there is a clear technical reason not to.
6. Ask for clarification when a requirement or architectural decision is genuinely ambiguous.
7. Implement the feature without unnecessarily rewriting unrelated code.
8. Run the relevant tests, type checks, and lint/build commands that are available.
9. Inspect the final diff.
10. Report what changed, what was tested, what failed, and what remains incomplete.

Coding agents must not invent product decisions, silently change the architecture, or replace existing implementations merely because a different approach is familiar.

If the code conflicts with the documentation, identify the discrepancy and propose a resolution rather than blindly forcing either side to match.

## 20. Definition of Done

A change is ready for review when:

- It satisfies the requested behavior.
- It follows the applicable code standards.
- Types and runtime validation are appropriate.
- Important error cases are handled.
- Relevant tests have been run.
- Available type checks, linting, and builds have been run as appropriate.
- No secrets or unrelated changes have been introduced.
- The final diff has been reviewed.
- Relevant documentation has been updated.
- Remaining limitations are reported honestly.

**Core rule:** Write code that fits Red Earth's existing architecture, protects authoritative game state, and remains understandable to the next developer who has to maintain it.