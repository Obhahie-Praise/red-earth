# AGENTS.md — Red Earth

## 1. Project Overview

Red Earth is a browser-first, online multiplayer first-person shooter inspired by Nigeria, Edo heritage, and Benin City.

Its visual direction is stylized realism with a distinctive red-earth identity. The project prioritizes responsive tactical gameplay, real-time multiplayer, browser accessibility, low-spec performance, and a strong cultural identity.

The current technology direction is:

- Next.js and React for the browser application.
- Three.js for 3D rendering.
- NestJS for backend services.
- Socket.IO for real-time communication.
- TypeScript for application code.
- pnpm workspaces and Turborepo for monorepo management.

Inspect the actual repository before assuming every planned technology or feature is fully configured.

## 2. Mandatory Working Rules

### Inspect Before Implementing

Before making changes:

1. Read this file and the root README.
2. Inspect the root directory and workspace configuration.
3. Read the relevant `package.json` files and existing scripts.
4. Inspect the existing implementation and related documentation.
5. Identify what already exists, what is missing, and what must change.
6. Prefer extending working implementations over replacing them.

Do not recreate existing files, configurations, components, or infrastructure without a clear technical reason.

Do not assume a feature is missing simply because it is not mentioned in the README.

### Preserve Existing Decisions

- Respect the established project structure.
- Preserve working configurations and existing dependencies.
- Do not relocate applications or rename packages without an explicit need.
- Do not replace the rendering engine, framework, networking library, or package manager without strong justification.
- Avoid unrelated refactors while implementing a requested feature.
- Do not overwrite established UI or visual styling unnecessarily.

If an important architectural decision is unclear, inspect the code and documentation first. Ask for clarification when the ambiguity cannot be resolved safely.

## 3. Monorepo and Tooling

- Use pnpm for package management.
- Use the existing Turborepo configuration for task orchestration.
- Respect the package boundaries defined in `pnpm-workspace.yaml`.
- Run commands from the appropriate workspace or use the existing root-level scripts.
- Keep dependencies in the package that actually uses them.
- Use existing package scripts instead of inventing parallel workflows.
- Avoid unnecessary dependencies and duplicated functionality.
- Do not introduce Docker or paid infrastructure unless explicitly required or justified by a concrete project need.

Before adding a dependency, check whether the functionality already exists in the codebase or can be implemented cleanly with the current stack.

## 4. TypeScript and Code Quality

- Use TypeScript for application code.
- Keep types explicit and meaningful.
- Avoid `any`; prefer proper interfaces, discriminated unions, generics, and `unknown` where appropriate.
- Validate untrusted external data at system boundaries.
- Handle errors intentionally instead of silently swallowing them.
- Avoid unnecessary abstractions and premature generalization.
- Keep functions and modules focused on clear responsibilities.
- Follow the existing linting, formatting, naming, and import conventions.

Do not disable compiler checks or weaken TypeScript configuration merely to make a build pass.

## 5. Frontend and Game Client

### Rendering

Three.js is the designated 3D rendering foundation.

- Keep rendering, simulation, input, networking, and UI responsibilities separated.
- Avoid unnecessary React re-renders in performance-sensitive game loops.
- Do not run the game simulation by repeatedly updating React component state.
- Manage animation frames, event listeners, controls, and GPU resources carefully.
- Dispose of geometries, materials, textures, render targets, and other resources when appropriate.
- Optimize draw calls, asset sizes, texture memory, and shader complexity.
- Use efficient asset loading and avoid blocking the initial application unnecessarily.

### Browser Accessibility and Performance

Red Earth should remain usable on a range of hardware.

- Avoid unnecessarily heavy dependencies and oversized assets.
- Provide loading, connection, and failure states.
- Handle browser focus, pointer lock, resizing, and tab visibility appropriately.
- Do not assume every device supports identical graphics capabilities.
- Use graceful fallbacks where practical.
- Profile before introducing expensive rendering techniques.

### UI and Visual Identity

Maintain a coherent visual identity built around stylized realism and red-earth-inspired colors.

- Prioritize readable HUD elements and responsive game controls.
- Keep menus, lobbies, settings, and other interface elements visually consistent.
- Avoid generic dashboard styling that conflicts with the game experience.
- Keep animation purposeful and performance-conscious.
- Do not invent unrelated visual themes or redesign established screens without a reason.

## 6. Backend and Multiplayer

NestJS is the backend framework, and Socket.IO is the real-time communication foundation.

### Server Authority

The server must remain authoritative over competitive game outcomes.

- Never trust the client to determine authoritative kills, damage, ammunition, rewards, currency, rankings, or match results.
- Validate client messages and gameplay actions.
- Enforce game rules and relevant rate limits on the server.
- Treat client-provided positions, actions, identifiers, and other payloads as untrusted input.
- Keep authentication and authorization separate from basic connection establishment.
- Prevent one player from modifying another player's private or authoritative state.

### Real-Time Communication

- Define explicit, typed event contracts.
- Validate incoming messages at the server boundary.
- Handle connection, disconnection, reconnection, room membership, and cleanup.
- Avoid broadcasting unnecessary data to every connected player.
- Use appropriate room-based communication and server-side access checks.
- Consider latency, update frequency, payload size, and server resource usage.
- Avoid assuming Socket.IO alone provides a complete game networking or simulation architecture.

### Game State and Simulation

Keep the following responsibilities clearly separated where the project grows to require them:

- Player sessions and identity.
- Matchmaking and game rooms.
- Match lifecycle and game modes.
- Authoritative gameplay simulation.
- Player movement and collision rules.
- Combat and damage resolution.
- Persistence and progression.

Do not implement every subsystem prematurely. Build the smallest coherent foundation needed for the current milestone.

## 7. Security and Configuration

- Never commit secrets or production credentials.
- Keep environment-specific settings outside source code.
- Validate and constrain external inputs.
- Use established authentication and password-handling practices.
- Apply authorization checks on protected operations.
- Avoid exposing internal errors or sensitive server details to clients.
- Use secure production transport and cookie settings where applicable.
- Document required environment variables and provide safe example values.

Do not add authentication providers, payment systems, or external services without a defined requirement.

## 8. Testing and Verification

After changes, run the checks relevant to the affected packages:

- TypeScript checks.
- Linting and formatting checks when configured.
- Unit and integration tests when available.
- Production builds for affected applications.
- Tests for networking contracts and gameplay rules where appropriate.

Use the actual scripts defined in the repository.

For multiplayer features, consider invalid payloads, disconnected clients, repeated events, unauthorized actions, and concurrent player interactions.

Do not claim that a check passed unless it was actually run and succeeded. Report failures and distinguish pre-existing problems from regressions introduced by the change.

## 9. Documentation and Progress Tracking

- Keep README instructions aligned with the actual repository.
- Document architectural decisions that materially affect future work.
- Document new environment variables, scripts, dependencies, and integration steps.
- Update existing project trackers or progress documents when present.
- Clearly distinguish implemented features from planned work.
- Record meaningful limitations and unresolved issues.
- Avoid creating redundant documentation files that duplicate existing material.

For substantial work, summarize completed changes, verification performed, remaining work, and any decisions requiring approval.

## 10. Cost and Infrastructure

The project should favor free or zero-cost development infrastructure where practical.

- Verify current free-tier limits before recommending a service.
- Account for networking, persistent connections, database usage, bandwidth, and storage.
- Avoid unnecessary paid services and subscriptions.
- Do not promise that a free tier can support a specific player count without capacity analysis and testing.
- Keep the architecture adaptable when a free-tier service becomes insufficient.

Free hosting must not be confused with unlimited capacity or guaranteed production reliability.

## 11. Scope and Product Integrity

The intended product includes online multiplayer, tactical FPS combat, Nigerian-inspired maps, multiple game modes, weapons, squads, communication, rankings, leaderboards, and cosmetic progression.

These are product goals, not evidence that the features already exist.

- Implement features incrementally.
- Keep the first playable experience ahead of nonessential complexity.
- Do not fabricate completed systems, test results, or performance figures.
- Avoid building elaborate infrastructure without a demonstrated requirement.
- Do not compromise server authority for implementation convenience.
- Keep cultural inspiration grounded and respectful.

## 12. Definition of Done

A change is complete when:

1. It satisfies the requested behavior.
2. It fits the existing architecture.
3. Relevant validation and error handling are implemented.
4. Appropriate checks have been run.
5. No unrelated configuration or UI changes have been introduced.
6. Necessary documentation has been updated.
7. Remaining limitations and follow-up work are reported.

For larger tasks, first present a concise implementation plan after inspecting the codebase. Then implement, verify, and summarize the actual results.

**Primary principle:** Build Red Earth as a coherent, performant multiplayer game. Inspect first, reuse what works, keep the architecture understandable, and introduce complexity only when the game needs it.