# Red Earth — Monorepo Structure

**Status:** Draft  
**Purpose:** Define repository organization, workspace boundaries, dependency rules and development workflows.

## 1. Overview

Red Earth uses a pnpm workspace managed by Turborepo.

The monorepo keeps the browser application, multiplayer server and genuinely shared code in one repository. This makes coordinated development easier while preserving clear boundaries between rendering, gameplay, networking and persistence.

The repository's actual files and configuration take precedence over examples in this document. Any difference between the intended structure and the current implementation must be recorded and resolved deliberately.

## 2. Intended Repository Structure

```text
red-earth/
├── apps/
│   ├── web/                  # Browser application and Three.js client
│   └── server/               # NestJS multiplayer server
│
├── packages/
│   ├── contracts/            # Shared event contracts and validation
│   ├── game-types/           # Shared gameplay types and constants
│   └── ...                   # Additional packages only when justified
│
├── docs/
│   ├── README.md
│   ├── product/
│   ├── architecture/
│   ├── development/
│   └── roadmap/
│
├── package.json
├── pnpm-workspace.yaml
├── pnpm-lock.yaml
├── turbo.json
├── tsconfig.json             # Only if the repository uses a root config
├── .gitignore
├── .npmrc                    # Only if repository settings require it
└── README.md
```

This is a target layout, not a claim that every listed package already exists.

Do not create empty packages merely to match the diagram. Create them when their responsibilities and dependencies are clear.

## 3. Workspace Responsibilities

### 3.1 `apps/web`

Contains the browser-facing application.

Responsibilities:

- Next.js routes and layouts.
- Player-facing interface components.
- Lobby and customization screens.
- Three.js scene and runtime integration.
- Client-side input and presentation logic.
- Socket.IO client integration.
- Browser-specific asset loading and resource management.

The web application should not contain the authoritative implementation of combat, matchmaking permissions, official scores or administrative decisions.

### 3.2 `apps/server`

Contains the NestJS backend.

Responsibilities:

- HTTP API endpoints.
- Socket.IO gateways.
- Session and connection handling.
- Match and squad lifecycle.
- Authoritative gameplay logic.
- Combat validation.
- Chat and moderation.
- Persistence and administrative services.

Keep modules organized around clear responsibilities. Avoid putting all application behavior into one gateway, controller or service.

### 3.3 Shared packages

Shared packages exist to prevent duplicate contracts and incompatible assumptions between the client and server.

Potential packages include:

**`packages/contracts`**

- Socket event names and payload schemas.
- Shared request and response contracts.
- Runtime validation schemas.
- Versioned protocol definitions where needed.

**`packages/game-types`**

- Shared gameplay identifiers.
- Weapon category definitions.
- Game mode identifiers.
- Small shared data types and constants.

A package should have one clear purpose. If contracts and gameplay types remain small and tightly related, a single shared package may be more appropriate initially.

Avoid importing server implementation details into the browser. Shared packages must not expose secrets, database internals or server-only dependencies.

## 4. Dependency Rules

The intended dependency direction is:

```text
apps/web ──────┐
               ├──> Shared packages
apps/server ───┘
```

Additional rules:

1. Applications may depend on shared packages.
2. Shared packages must not depend on either application.
3. Browser code must not import server-only modules.
4. Server code must not depend on browser-only APIs.
5. Shared code must remain safe to import in its intended runtime.
6. Avoid circular dependencies.
7. Use explicit package exports instead of importing arbitrary internal files across package boundaries.
8. Add dependencies to the workspace that actually uses them.

Do not add a dependency to the repository root just because it is convenient. Root dependencies should support repository-wide tooling or scripts.

## 5. Package Management

Use pnpm as the package manager.

Requirements:

- Keep `pnpm-lock.yaml` under version control.
- Install dependencies using the repository's configured pnpm version.
- Use workspace references for internal packages.
- Avoid mixing npm, Yarn and pnpm lockfiles.
- Do not edit the lockfile manually.
- Use reproducible installation commands in CI.
- Verify the installed Node.js and pnpm versions against the repository's engine requirements.

When adding an internal package dependency, use the workspace protocol where appropriate, for example:

```json
{
  "dependencies": {
    "@red-earth/contracts": "workspace:*"
  }
}
```

The example illustrates the intended convention. The package name and dependency must match the actual workspace configuration.

## 6. Turborepo Responsibilities

Turborepo coordinates tasks across the workspace.

The repository should define tasks appropriate to its actual applications and packages, such as:

- `dev`
- `build`
- `lint`
- `typecheck`
- `test`

Not every package must implement every task. Task configuration should reflect real package scripts and dependency relationships.

Build tasks may be cached when their inputs and outputs are correctly declared. Development servers should not be treated as ordinary cacheable build outputs.

Before changing `turbo.json`, inspect the existing configuration and preserve working task behavior unless a change is necessary.

## 7. TypeScript Standards

Use TypeScript across the applications and shared packages.

Requirements:

- Enable strict checking where compatible with the existing setup.
- Avoid `any`; use precise types, `unknown` with validation, or properly defined interfaces.
- Keep browser and server compiler settings appropriate to their runtimes.
- Avoid weakening type checks to hide implementation problems.
- Validate untrusted network data at runtime. TypeScript types alone do not validate incoming messages.
- Keep shared contracts explicit and stable.

Do not force one TypeScript configuration across every package if their runtime requirements differ.

## 8. Environment Configuration

Environment-specific values must not be committed as secrets.

Requirements:

- Keep local environment files out of version control.
- Provide safe example environment files where useful.
- Validate required environment variables during startup.
- Distinguish public browser variables from server-only secrets.
- Never expose server credentials through client-side environment variables.
- Document how to configure development and production environments.

A variable intended for the browser must be considered public, even if its name appears obscure.

## 9. Local Development Workflow

The intended workflow is:

1. Install the repository's required Node.js and pnpm versions.
2. Install dependencies from the repository root.
3. Configure required local environment variables.
4. Start the applications using the existing workspace scripts.
5. Verify that the web application and server start successfully.
6. Run the relevant type checks, linting and tests.
7. Test multiplayer behavior with multiple browser clients when the feature requires it.

The actual commands must be copied from and verified against the root and application `package.json` files. Do not invent scripts that the repository does not define.

## 10. Adding Dependencies

Before adding a dependency:

1. Check whether the required capability already exists in the repository.
2. Consider native platform APIs or existing dependencies.
3. Check browser compatibility and server runtime compatibility.
4. Consider package size, maintenance status and licensing.
5. Evaluate the effect on build time, bundle size and operational cost.
6. Install the dependency in the correct workspace.
7. Run relevant type checks, builds and tests.

Prefer the smallest suitable dependency set. Avoid unnecessary frameworks and overlapping libraries.

## 11. Code Organization

Organize code around clear responsibilities rather than creating large generic utility folders.

Examples of useful boundaries include:

- Rendering and scene management.
- Player input and movement presentation.
- Networking adapters.
- Game modes and match state.
- Weapon definitions and combat validation.
- Lobby and social interfaces.
- Chat and moderation.
- Persistence and account services.

Names and folder locations should follow existing project conventions unless a deliberate refactor is approved.

Avoid broad rewrites merely to make the directory tree look cleaner.

## 12. Documentation and Progress Tracking

Documentation lives in `docs/`.

- `docs/product/` defines intended product behavior.
- `docs/architecture/` defines system boundaries and technical constraints.
- `docs/development/` documents setup, standards and testing.
- `docs/roadmap/` records sequencing, progress and decisions.

`docs/roadmap/PROGRESS.md` is the canonical implementation tracker.

After completing work, update the tracker with the actual result and relevant verification. Do not mark work complete merely because code was written.

## 13. Git and Repository Safety

All changes must remain within the intended repository.

Requirements:

- Inspect Git status before changing files.
- Preserve unrelated user changes.
- Do not initialize nested Git repositories without an explicit reason.
- Do not delete Git metadata or rewrite history as part of ordinary development.
- Do not force-push, reset or discard changes without explicit authorization.
- Keep secrets, generated output and local-only configuration out of version control.
- Review the final diff before committing.

Repository cleanup must never modify other projects or directories outside the authorized scope.

## 14. Validation Before Completion

Before considering a monorepo-level change complete:

- Verify the affected workspace packages.
- Run applicable type checks and tests.
- Confirm package imports resolve correctly.
- Check for circular dependencies where relevant.
- Confirm root scripts still work.
- Inspect the final diff.
- Update affected documentation and the progress tracker.

## 15. Definition of Completion

The monorepo structure is ready to support implementation when:

- Workspace configuration is valid.
- Applications and shared packages have clear boundaries.
- Internal imports resolve through declared package dependencies.
- Build, test and development scripts match actual package scripts.
- Environment configuration is documented.
- Repository changes preserve existing user work.
- The documentation matches the real repository.