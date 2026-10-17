# Development Setup

**Project:** Red Earth  
**Path:** `docs/development/SETUP.md`  
**Status:** Draft

## 1. Purpose

This guide explains how to prepare a local development environment, install dependencies, configure environment variables, run Red Earth, and troubleshoot common setup problems.

The repository is the source of truth. Always check its existing configuration before adding dependencies, changing scripts, or creating new files.

## 2. Technology Stack

Red Earth uses the following intended stack:

- **Package manager:** pnpm
- **Monorepo orchestration:** Turborepo
- **Web application:** Next.js, React, TypeScript
- **3D rendering:** Three.js
- **Game server:** NestJS with Socket.IO, subject to confirmation against the current implementation
- **Database:** Neon PostgreSQL
- **ORM and migrations:** Drizzle ORM
- **Game asset delivery:** Render Static Site
- **Game server hosting:** Render Web Service

The exact installed versions and available commands must be taken from the repository's manifests and configuration files.

## 3. Prerequisites

Install the following before starting development:

- Node.js version compatible with the repository's `engines` field and tooling.
- pnpm version compatible with the repository's `packageManager` field.
- Git.
- A code editor, preferably VS Code.
- A modern browser with WebGL support.

A local PostgreSQL installation is not automatically required because the project uses Neon. A separate local or test database may be useful for isolated integration tests.

### Verify the installation

Run these commands in PowerShell:

```powershell
node --version
pnpm --version
git --version
```

If pnpm is unavailable, enable Corepack if supported by the installed Node.js version:

```powershell
corepack enable
```

Then activate the pnpm version specified by the repository rather than installing an arbitrary global version.

If Corepack is unavailable or the command fails, use the installation method documented for the project's Node.js and pnpm versions.

## 4. Clone and Install

If the repository has not yet been cloned:

```powershell
git clone <REPOSITORY_URL>
cd <REPOSITORY_DIRECTORY>
```

Replace the placeholders with the actual repository URL and directory.

If you already have the repository locally, open its existing directory instead. Do not clone a second copy unnecessarily.

Install dependencies from the monorepo root:

```powershell
pnpm install
```

Run installation from the directory containing the root `package.json` and `pnpm-workspace.yaml`.

Do not install workspace dependencies independently inside each application unless the repository explicitly requires it.

## 5. Understand the Repository Before Changing It

Before starting development, inspect the existing files:

```powershell
Get-Content package.json
Get-Content pnpm-workspace.yaml
Get-ChildItem
```

Inspect the application manifests and configuration files as needed.

Confirm:

- Which workspace contains the web application.
- Which workspace contains the game server.
- Which shared packages already exist.
- Which development, build, test, and type-check scripts are available.
- How environment variables are loaded.
- Which database migration commands are configured.
- Whether the multiplayer implementation uses NestJS with Socket.IO, Colyseus, or another existing integration.

Do not create duplicate applications, shared packages, or configuration files because an example in this document does not match the current repository.

## 6. Environment Variables

Environment variables contain configuration that differs between local development, testing, and production.

Check the existing environment examples and configuration schemas before creating or modifying any environment file.

The following values are examples of configuration the project may require. They are not a claim that every variable already exists in the codebase.

| Configuration | Purpose |
|---|---|
| Database connection URL | Connects Drizzle to the appropriate Neon PostgreSQL database |
| Authentication secrets | Supports the configured authentication and session mechanism, if implemented |
| Server port | Controls the local game-server listener |
| Public API or server URL | Lets the web client connect to the correct local server |
| Allowed origins | Restricts cross-origin requests where applicable |
| Asset base URL | Points to the configured static asset location |

Use the exact variable names expected by the existing application. For example, do not assume that the database variable is named `DATABASE_URL` without checking the code.

### Local environment files

If the project provides example files, copy them using PowerShell. For example:

```powershell
Copy-Item .env.example .env
```

Only run this if that example file exists and is appropriate for the application being configured.

If the web app and server use separate environment files, follow their respective instructions.

Important rules:

- Never commit populated `.env` files or secrets.
- Keep production credentials out of local development.
- Use separate credentials for development and testing.
- Keep public client-side variables free of secrets.
- Do not expose database connection strings or server secrets through browser bundles.
- Never use production credentials in destructive tests.

## 7. Configure Neon and Drizzle

Red Earth uses Neon PostgreSQL for persistent relational data and Drizzle ORM for schema definitions, queries, and migrations.

Before running database commands:

1. Confirm the existing Drizzle configuration and schema locations.
2. Configure the appropriate development database connection.
3. Verify that the selected database is not production.
4. Check the scripts available in the relevant workspace.
5. Follow the repository's migration workflow.

Inspect the available commands:

```powershell
pnpm run
```

You can also inspect the relevant workspace's `package.json` for its database scripts.

Use the project's configured migration command rather than guessing a command or creating a second migration workflow.

**Do not run migrations against production casually.** Review generated migrations, understand destructive changes, and follow the project's deployment procedure.

## 8. Start the Development Environment

Start the applications using the commands defined in the repository.

First, inspect the root scripts:

```powershell
pnpm run
```

If the root manifest provides a development command that starts all required applications, use that command.

Otherwise, run the relevant workspace commands in separate PowerShell terminals. For example, if the actual workspace names and scripts match these placeholders:

```powershell
pnpm --filter <WEB_WORKSPACE_NAME> dev
```

```powershell
pnpm --filter <SERVER_WORKSPACE_NAME> start:dev
```

Replace the placeholders with the actual workspace names. Do not paste these example commands unchanged.

Once started, confirm that:

- The web application opens in the browser.
- The game server starts without configuration errors.
- The web client can reach the intended server endpoint.
- Database-backed functionality can connect to the development database.
- The browser console and server logs contain no unresolved startup errors.

A running landing page alone does not prove that multiplayer connectivity or database access works.

## 9. Working with Game Assets

The configured Render Static Site is intended to deliver game assets such as models, textures, audio, maps, and animations.

For local development:

- Follow the existing repository's asset directory and loading conventions.
- Use local assets or the configured development asset URL as supported by the implementation.
- Verify that asset URLs resolve correctly in the browser.
- Check that missing assets produce useful errors rather than breaking the entire scene.
- Avoid committing large generated assets unless the repository's asset policy permits it.
- Keep persistent asset metadata and ownership information in the database where appropriate; the database is not the binary asset host.

Do not create a new storage provider or alter the asset-hosting arrangement without an explicit architecture decision.

## 10. Useful Development Checks

Use the existing scripts to run checks from the repository root.

```powershell
pnpm run
```

Check the available scripts before running commands for:

- Development servers.
- Production builds.
- Linting.
- Type-checking.
- Unit tests.
- Integration tests.
- Database migrations.
- Database schema generation.

If the repository uses Turborepo task filtering, follow its existing configuration and package names. Do not assume that every workspace implements every task.

Refer to `docs/development/CODE_STANDARDS.md` for coding conventions and `docs/development/TESTING.md` for testing expectations.

## 11. Common Problems

### pnpm is not recognized

Check whether Node.js and pnpm are installed and whether their executable directories are on `PATH`. Restart the terminal after changing environment configuration.

### Dependency installation fails

Verify the Node.js and pnpm versions required by the repository. Check the lockfile, registry access, and error output before changing dependencies.

Avoid deleting the lockfile or upgrading packages as a first response.

### The web client cannot connect to the game server

Check that the server is running, the client points to the correct local endpoint, and the configured origins permit the connection.

For Socket.IO, verify the configured transport and connection path. If the repository uses another multiplayer framework, follow its actual connection configuration.

### Database connection fails

Check the database URL, credentials, network connectivity, and whether the correct Neon database is being used. Confirm that required migrations have been applied.

Never print the full database URL or credentials when sharing logs.

### Assets fail to load

Check the asset base URL, file paths, response status, and browser network panel. Confirm that the referenced files exist and that the configured host serves them correctly.

### TypeScript or workspace errors appear

Check workspace dependencies and package exports. Run the relevant type-check command and inspect the first meaningful error before changing shared configuration.

### The game runs slowly

Check browser console errors, asset sizes, scene complexity, and frame rate. Test on representative lower-spec hardware before assuming the problem is caused by the server.

## 12. Safe Development Practices

- Work on a feature branch when making substantial changes.
- Keep `.env` files and secrets out of version control.
- Avoid destructive commands until their effects are understood.
- Never test account deletion, database resets, or destructive migrations against production.
- Do not introduce dependencies without a clear need.
- Run relevant tests before committing changes.
- Keep documentation aligned with actual scripts and architecture.
- Record unresolved setup problems rather than hiding them with speculative configuration changes.

## 13. Setup Completion Checklist

- [ ] Required Node.js and pnpm versions are installed.
- [ ] Repository dependencies install successfully.
- [ ] Existing workspace structure and scripts have been inspected.
- [ ] Required local environment variables are configured.
- [ ] Development database connectivity is verified.
- [ ] Required migrations are applied to the development database.
- [ ] Web application starts successfully.
- [ ] Game server starts successfully.
- [ ] Client-to-server connectivity is verified.
- [ ] Essential game assets load successfully.
- [ ] Relevant lint, type-check, and test commands run successfully.

## 14. Keeping This Guide Accurate

Update this document whenever setup requirements, workspace scripts, environment variables, database workflows, or development commands change.

Commands copied into this guide must be verified against the repository before being presented as ready to run. This document describes the intended workflow; the current codebase determines the exact commands and configuration names.