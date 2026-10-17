# Red Earth — Data Model and Storage Architecture

**Path:** `docs/architecture/DATA-MODEL.md`  
**Status:** Draft — canonical data architecture  
**Database:** Neon PostgreSQL  
**ORM:** Drizzle ORM  
**Live game server:** NestJS + Socket.IO on Render  
**Game asset delivery:** Render Static Site  
**Monorepo:** pnpm + Turborepo

---

## 1. Purpose

This document defines how Red Earth stores, relates, validates, and retrieves its data.

It covers the complete planned game data model, including:

- Accounts, authentication, and player profiles.
- Characters, weapons, skins, and cosmetic assets.
- Ownership, equipment, currency, and purchases.
- Player progression, XP, ranks, and competitive ratings.
- Match history, statistics, leaderboards, and seasons.
- Friends, presence, parties, invitations, and squads.
- Lobby and match-related chat.
- Reports, moderation, bans, and administrative audit logs.
- Game configuration, maps, modes, and asset manifests.
- Database migrations, persistence, caching, and operational responsibilities.

The aim is to establish a model that supports the intended game without creating unnecessary complexity or duplicating the same information across unrelated systems.

This is the target data architecture. Existing repository files and implemented features must be inspected before assuming any table or workflow already exists.

## 2. Infrastructure Responsibilities

Each infrastructure component has a distinct responsibility.

| Infrastructure | Responsibility | Data it owns |
|---|---|---|
| Neon PostgreSQL | Durable relational storage | Accounts, profiles, inventory, progression, results, rankings, social data, moderation |
| Drizzle ORM | Database access and schema management | Typed schema definitions, queries, relations, migrations |
| Render Web Service | Persistent real-time server process | Live game simulation, connections, active rooms, temporary match state |
| Render Static Site | Static asset delivery | Public game models, textures, maps, animations, audio, and other game files |
| Browser memory | Temporary client state | Current rendering state, input, visual effects, UI state, local interpolation |
| Shared TypeScript packages | Shared contracts and definitions | Types, validation schemas, identifiers, and protocol contracts—not authoritative persistent data |

### 2.1 Neon PostgreSQL

Neon is the persistent source of truth for account-owned and long-lived game data.

Use it for information that must survive browser refreshes, server restarts, disconnections, and completed matches.

Examples include a player's owned skins, account level, ranked rating, match history, and currency balance.

### 2.2 Render Web Service

The real-time server owns live match simulation.

Examples include player positions, current health, ammunition during a match, active projectiles, temporary combat events, and current match timers.

These values should normally remain in server memory while a match is running. Persist match results and other important changes at appropriate transaction boundaries.

Do not write every movement update or simulation tick to PostgreSQL.

If the server restarts unexpectedly, the recovery behavior must be defined for each game mode. Do not assume in-memory match state can be recovered after a process restart.

### 2.3 Render Static Site

Static hosting delivers game assets; it does not establish ownership or permissions.

It may host:

- Character models and animations.
- Weapon models and inspect animations.
- Character and weapon skins.
- Map geometry and environment assets.
- Textures, materials, and visual effects.
- Sound effects and music.
- UI icons and other game artwork.

The database stores metadata and references to these files, not the binary files themselves.

Only public assets should be exposed through public static URLs. Private or restricted assets would need a different access strategy if introduced later.

### 2.4 What we are not doing

- Do not store large GLB, texture, audio, or map files as PostgreSQL blobs.
- Do not use Neon as a real-time movement or snapshot store.
- Do not treat browser state as authoritative inventory or currency.
- Do not introduce Redis, a separate object-storage service, or another database without a demonstrated need.
- Do not create duplicate account, inventory, or ranking records in multiple places without a clear synchronization strategy.

---

## 3. Data Model Overview

The model is organized into logical domains.

```text
Identity
├── Users
├── Accounts / Authentication
├── Sessions
└── Player Profiles

Game Catalog
├── Game Modes
├── Maps
├── Characters
├── Weapons
├── Cosmetics
├── Asset Manifest
└── Loadout Rules

Ownership and Progression
├── Player Inventory
├── Currency Accounts
├── Currency Transactions
├── XP Transactions
├── Progression
├── Achievements
└── Equipment / Loadouts

Competitive Play
├── Matches
├── Match Participants
├── Player Statistics
├── Ranked Ratings
├── Seasons
├── Rating History
└── Leaderboard Results

Social
├── Friendships
├── Parties / Squads
├── Invitations
├── Player Blocks
└── Chat Messages

Trust and Operations
├── Reports
├── Moderation Actions
├── Bans / Restrictions
├── Admin Audit Logs
└── Game Configuration
```

Use relational constraints to protect data integrity. Do not rely exclusively on application code to prevent duplicate ownership records, invalid foreign keys, or inconsistent currency transactions.

## 4. Identity and Player Profiles

### 4.1 Users

The user record represents the account identity.

Proposed fields:

- `id` — primary identifier, preferably a UUID.
- `username` — unique normalized username.
- `displayName` — player-facing name.
- `email` — nullable where the authentication model permits it; unique when present and verified according to the chosen policy.
- `emailVerifiedAt` — verification timestamp, if applicable.
- `avatarAssetId` — optional reference to an approved avatar asset.
- `status` — account status such as active or suspended.
- `createdAt`
- `updatedAt`
- `deletedAt` — nullable deletion or anonymization marker, if required.

Keep authentication credentials and provider-specific identity records in the authentication system's appropriate tables. Do not create a second password system alongside an established authentication library.

If Better Auth or another authentication solution is used, inspect its schema and integrate with it instead of duplicating its user and session records.

### 4.2 Player profiles

A profile stores public-facing gameplay identity and selected preferences.

Proposed fields:

- `userId` — primary key and foreign key to the user.
- `bio` — optional, length-limited description.
- `countryCode` — optional and user-controlled.
- `preferredCharacterId` — optional default character.
- `createdAt`
- `updatedAt`

Profile information should not be mixed with rapidly changing match state.

Do not expose email addresses, authentication metadata, moderation notes, or other private fields in public player-profile responses.

### 4.3 Player statistics

Keep durable aggregate statistics in a dedicated record or a well-defined statistics model.

Potential fields include:

- Total matches played.
- Wins and losses.
- Eliminations and deaths.
- Assists.
- Total match time.
- Best match result.
- Current and best competitive rating, where appropriate.

Statistics should be updated from validated server-side match results. They must not be directly editable by the player.

Where statistics can be derived reliably from match history, document which values are cached aggregates and how they are rebuilt.

---

## 5. Game Catalog and Asset Metadata

The game catalog describes which content exists. The static files contain the actual content.

### 5.1 Characters

A character definition may contain:

- `id`
- `slug`
- `name`
- `description`
- `baseAssetId`
- `defaultSkinId`
- `isDefault`
- `isAvailable`
- `createdAt`
- `updatedAt`

Red Earth targets at least five playable characters for the intended content scope.

Character gameplay attributes must be separate from cosmetic appearance. Cosmetic selection must not silently change competitive statistics.

If characters eventually have different gameplay attributes, define those rules explicitly and ensure they are balanced and validated by the server.

### 5.2 Weapons

A weapon definition may contain:

- `id`
- `slug`
- `name`
- `weaponType`
- `baseAssetId`
- `configuration`
- `isAvailable`
- `createdAt`
- `updatedAt`

The intended initial weapon roster is:

- Three assault rifles.
- Three SMGs.
- Two shotguns.
- Grenades.
- Flashbangs.

Weapon configuration can describe base damage, fire rate, ammunition capacity, reload duration, recoil parameters, projectile behavior, and other gameplay settings.

Use a validated, versioned configuration format. Do not let arbitrary database JSON bypass server-side gameplay validation.

The authoritative server must determine how weapon configuration is interpreted. A browser must not be able to grant itself better weapon statistics by modifying local files.

### 5.3 Cosmetics

Cosmetics represent appearance-only items.

Examples:

- Character skins.
- Weapon skins.
- Character accessories.
- Emotes.
- Banners, profile frames, or other approved presentation items.

Proposed fields:

- `id`
- `slug`
- `name`
- `description`
- `cosmeticType`
- `targetType`
- `targetId` — nullable when a cosmetic can apply to multiple targets.
- `assetId`
- `rarity` — presentation classification, not proof of value or gameplay strength.
- `acquisitionType`
- `isAvailable`
- `releaseAt` — optional.
- `retireAt` — optional.
- `createdAt`
- `updatedAt`

Cosmetic definitions describe what an item is. They do not prove that a player owns it.

### 5.4 Assets

Create an asset manifest so the game can resolve asset references consistently.

Proposed fields:

- `id`
- `assetKey` — unique stable identifier.
- `assetType` — model, texture, audio, animation, map, icon, or other defined type.
- `url` — static asset URL or path.
- `contentHash` — optional checksum for integrity and cache invalidation.
- `fileSizeBytes` — optional.
- `mimeType`
- `version`
- `isAvailable`
- `createdAt`
- `updatedAt`

The manifest stores metadata, not the file contents.

Use stable asset keys in game definitions. Store versioned URLs or paths where necessary so changes to a model or texture do not cause stale browser caches to serve incompatible files.

Do not put temporary development URLs into production catalog records.

### 5.5 Maps and modes

A map definition may contain:

- `id`
- `slug`
- `name`
- `description`
- `assetId` or references to a group of map assets.
- `supportedModes`
- `configuration`
- `isAvailable`

A mode definition may contain:

- `id`
- `slug`
- `name`
- `description`
- `configuration`
- `isAvailable`

The initial content target is at least five maps and at least three game modes.

Actual collision geometry, spawn points, objectives, and gameplay rules must be validated by the game server or the trusted simulation configuration. Static map assets are not themselves proof of a valid gameplay state.

---

## 6. Ownership, Inventory, and Equipment

Ownership is a central game rule. It must be explicit and server-controlled.

### 6.1 Inventory

Use a player-inventory table to represent ownership of unique cosmetic or unlockable items.

Proposed fields:

- `id`
- `userId`
- `itemType`
- `itemId`
- `acquisitionSource`
- `acquisitionReference` — optional transaction, reward, or grant reference.
- `acquiredAt`

Enforce uniqueness for the combination that identifies a player's owned item. For unique cosmetics, one player should not acquire duplicate ownership rows for the same item.

If stackable consumables or duplicate-convertible items are introduced later, model their quantity and rules explicitly rather than weakening ownership constraints for every item.

The catalog answers, "What items exist?" Inventory answers, "Which items does this player own?"

### 6.2 Default ownership

Default characters, starter skins, and complimentary skins must be represented by explicit game rules.

Two supported approaches are:

1. Grant starter items to a player's inventory when their profile is initialized.
2. Define universal starter entitlements and evaluate them through a centralized ownership service.

Choose one consistent strategy. Do not mix implicit ownership checks throughout the codebase.

If starter items are granted as inventory rows, make the operation idempotent so retries cannot duplicate grants.

### 6.3 Equipment and loadouts

Use a dedicated loadout model rather than treating ownership as equipment.

Proposed tables:

- `player_loadouts`
- `player_loadout_items`

A loadout may contain:

- `id`
- `userId`
- `name`
- `isActive`
- `createdAt`
- `updatedAt`

Loadout items may contain:

- `loadoutId`
- `slot`
- `itemType`
- `itemId`

The server must verify that every equipped item is owned or otherwise legitimately available, compatible with the selected slot, and allowed in the current mode.

The active character and weapon appearance may be loaded into the lobby preview, but the client cannot use that preview to authorize an item in a match.

---

## 7. Currency and the Economy

Red Earth may include an in-game currency and unlockable cosmetics. Currency must be protected against duplication and unauthorized modification.

### 7.1 Currency accounts

A currency account represents a player's balance for a specific currency.

Proposed fields:

- `id`
- `userId`
- `currencyCode`
- `balance`
- `updatedAt`

Enforce one account per player and currency.

Use integer values for currency units wherever possible. Do not use floating-point arithmetic for balances.

### 7.2 Currency transactions

Every balance change must have a corresponding durable transaction record.

Proposed fields:

- `id`
- `currencyAccountId`
- `transactionType`
- `amount`
- `balanceAfter` — optional but useful for auditing.
- `referenceType`
- `referenceId`
- `idempotencyKey`
- `createdAt`

Examples of transaction types:

- Match reward.
- Achievement reward.
- Event reward.
- Cosmetic purchase.
- Administrative grant.
- Refund or correction.

The server must create the transaction and update the balance atomically in a PostgreSQL transaction.

Use a unique idempotency key for operations that may be retried. Repeating the same reward or purchase request must not grant currency twice.

Never accept a new balance directly from the client.

### 7.3 Purchases

A cosmetic purchase should follow this sequence:

1. Authenticate the player.
2. Verify that the item exists and is available for purchase.
3. Check the price and purchase rules using trusted server data.
4. Begin a database transaction.
5. Verify sufficient balance.
6. Deduct currency and record the transaction.
7. Grant inventory ownership.
8. Commit the transaction.

If any step fails, roll back the transaction.

A unique ownership constraint and idempotency mechanism must prevent retries from charging a player twice or granting duplicate items.

### 7.4 Free and complimentary items

Free starter items should not require a currency transaction unless there is a meaningful reason to record a grant.

Limited-time events and promotional rewards should have explicit eligibility and idempotency rules.

Paid real-money purchases are not assumed by this document. If introduced later, they will require a separate payment and entitlement design.

---

## 8. XP and Player Progression

XP represents long-term player progression and is distinct from ranked skill.

### 8.1 Progression record

Proposed fields:

- `userId`
- `level`
- `totalXp`
- `updatedAt`

A player's level should be calculated from total XP using a centralized progression curve, or stored and updated transactionally with total XP.

Do not implement conflicting level formulas in the client and server.

### 8.2 XP transactions

Use a ledger for significant XP changes.

Proposed fields:

- `id`
- `userId`
- `amount`
- `sourceType`
- `sourceId`
- `idempotencyKey`
- `createdAt`

Possible sources include completed matches, achievements, and approved events.

Award XP only after the server validates the relevant result. Repeated result processing must not grant XP more than once.

The ledger allows the team to investigate rewards and rebuild progression if an aggregate becomes inconsistent.

### 8.3 Achievements

A minimal achievement system can use:

- `achievements` — definitions and reward rules.
- `player_achievements` — completion state and timestamps.

Achievements must be evaluated against trusted game events or validated persistent statistics.

The data model should support new achievements without requiring a new database table for every individual achievement.

---

## 9. Competitive Rankings

**Player level and competitive rating are different systems.**

Level measures accumulated progression. Competitive rating estimates skill within a ranked mode or competitive queue.

Do not use total XP as the sole ranking formula.

### 9.1 Ranked rating records

Use a rating record scoped to the player, competitive mode or queue, and season.

Proposed fields:

- `id`
- `userId`
- `modeId`
- `seasonId`
- `rating`
- `gamesPlayed`
- `wins`
- `losses`
- `provisional`
- `updatedAt`

Enforce uniqueness across the player, ranked mode, and season.

This permits a player to have different ratings for different competitive modes and seasons.

### 9.2 Recommended initial rating model

For the first ranked implementation, use an Elo-style rating model. It is relatively easy to explain, test, and maintain.

For a match between two players or two team rating groups:

\[
E_A=\frac{1}{1+10^{(R_B-R_A)/400}}
\]

\[
R'_A=R_A+K(S_A-E_A)
\]

Where:

- \(R_A\) is the current rating of side A.
- \(R_B\) is the opposing rating.
- \(E_A\) is the expected result for side A.
- \(S_A\) is the actual result: 1 for a win, 0.5 for a draw, or 0 for a loss.
- \(K\) controls how quickly the rating changes.

For team matches, calculate the expected result using an agreed team-rating aggregation method. Update individual players from the team's result, with the exact policy documented and tested.

Use a higher \(K\) for provisional players and a lower \(K\) after enough matches, if testing supports that policy.

The exact initial rating, \(K\) values, placement rules, and treatment of draws remain configuration decisions. Do not hard-code them across multiple services.

This formula is a starting point for ranked play, not a claim that Elo is the only suitable competitive system. Reassess it after observing real match data.

### 9.3 What should affect rating?

For the initial ranked system, use the verified match outcome and the rating difference between opponents.

Do not directly reward raw eliminations as the main source of rating changes. That can encourage players to chase kills instead of playing objectives.

If individual performance weighting is introduced later, it must be mode-specific, tested against exploitative strategies, and documented separately.

### 9.4 Match processing and idempotency

When a match ends:

1. The authoritative game server determines the final result.
2. The server validates participating players and match completion.
3. The result is persisted with a unique match identifier.
4. Rating changes are calculated from the pre-match ratings.
5. Rating history and current rating are updated atomically.
6. XP, statistics, and eligible rewards are processed with idempotency protection.

A match must not award progression or competitive rating twice if a save or retry occurs.

For team matches, update all eligible players consistently. Avoid partial rating updates when one database operation fails.

### 9.5 Ranked tiers

Display tiers should be mapped from numeric rating ranges through centralized configuration.

Example tier categories might include Bronze, Silver, Gold, Platinum, Diamond, and a top competitive tier.

These are proposed labels, not a finalized Red Earth ranking ladder.

Define:

- Tier thresholds.
- Division boundaries, if used.
- Promotion and demotion behavior.
- Placement matches.
- Season reset or soft-reset rules.
- Tie-breaking and display behavior.

Store the underlying numeric rating independently of the display tier. This allows thresholds and names to change without rewriting every player's match history.

---

## 10. Leaderboards

Leaderboards are public competitive views derived from trusted game data.

They must not be editable by clients.

### 10.1 Leaderboard types

The model should support:

- Ranked rating by mode and season.
- Seasonal wins.
- Seasonal match performance.
- Career wins.
- Career eliminations.
- Other clearly defined statistics when useful.

Avoid publishing a large number of leaderboards that have no clear gameplay purpose.

### 10.2 Source of truth

Use the following sources:

| Leaderboard | Source data |
|---|---|
| Ranked rating | Current rating records |
| Seasonal wins | Validated match participants and results |
| Career wins | Validated match history or verified aggregate statistics |
| Career eliminations | Validated match statistics |
| Seasonal placement | Season-scoped rating or statistics records |

The browser must never submit its own leaderboard score.

### 10.3 How leaderboard data is built

For the initial version, do not introduce a separate analytics database.

Use Neon PostgreSQL as the source of truth and build leaderboard queries from the rating and statistics tables.

For small datasets, direct indexed queries are sufficient.

For example, a ranked leaderboard can:

1. Filter ratings by mode and season.
2. Exclude players who are ineligible for public ranking.
3. Join the public profile fields needed for display.
4. Sort by rating descending.
5. Apply a documented tie-breaking rule.
6. Return a bounded page of results.

A seasonal wins leaderboard can aggregate eligible completed match participants for the chosen season, or use a verified aggregate that is updated transactionally.

### 10.4 Aggregate statistics

As match volume grows, maintain aggregate statistics rather than scanning every match on every leaderboard request.

A match result can update:

- Total matches.
- Wins and losses.
- Eliminations and deaths.
- Assists.
- Time played.
- Seasonal statistics.

Use transactions and idempotency to prevent double-counting.

Keep match history as the durable record and aggregates as derived data that can be rebuilt or reconciled.

### 10.5 Caching and snapshots

Initially, indexed SQL queries and short-lived application caching may be enough.

If leaderboard queries become expensive:

- Add suitable PostgreSQL indexes.
- Cache common leaderboard pages for a short period.
- Introduce scheduled leaderboard snapshots if needed.
- Invalidate or refresh affected data after match results are committed.

Do not introduce another storage service solely because leaderboards exist.

### 10.6 Eligibility and tie-breaking

Define eligibility rules before publishing a competitive leaderboard.

Potential requirements include minimum ranked matches, account standing, and season participation.

A tie-breaking policy should be deterministic. For example:

1. Higher rating.
2. More eligible ranked wins.
3. Earlier achievement of the tied rating, if tracked and desired.

The final policy should be consistent across API responses and displayed rankings.

---

## 11. Matches and Match History

### 11.1 Matches

The `matches` table represents completed or otherwise durably recorded match sessions.

Proposed fields:

- `id`
- `modeId`
- `mapId`
- `seasonId` — nullable for unranked matches or modes without seasons.
- `status`
- `startedAt`
- `endedAt`
- `durationSeconds`
- `serverVersion`
- `resultSummary`
- `createdAt`

Use explicit match statuses such as completed, aborted, or cancelled. Do not mark an interrupted match as a valid win merely because the process stopped.

The exact lifecycle may include additional temporary states in server memory. Only states that need durable storage belong in the database.

### 11.2 Match participants

Use `match_participants` to associate players with matches.

Potential fields:

- `matchId`
- `userId`
- `teamId` — optional.
- `placement` — nullable where the mode has no placement system.
- `outcome`
- `kills`
- `deaths`
- `assists`
- `damageDealt`
- `score`
- `timePlayedSeconds`
- `ratingBefore` — optional for ranked matches.
- `ratingAfter` — optional for ranked matches.
- `createdAt`

Use a unique constraint on `(matchId, userId)` unless a documented game mode requires a different participant model.

Store only statistics the game can reliably calculate and the product has a reason to display or analyze.

### 11.3 Match result integrity

A completed match must be persisted through an idempotent server-controlled workflow.

If the process retries result persistence, it must not create a duplicate match, participant, reward, or rating change.

Consider using a unique match identifier generated before finalization and a durable processing status to recover from interrupted result processing.

Do not write per-frame player positions or every combat event to PostgreSQL by default. Detailed replay storage, if desired later, requires a separate design with explicit volume and retention limits.

---

## 12. Seasons and Competitive Resets

Use a `seasons` table to represent ranked seasons and seasonal leaderboards.

Proposed fields:

- `id`
- `name`
- `startsAt`
- `endsAt`
- `status`
- `configuration`
- `createdAt`

A season determines the period used for seasonal rankings, rewards, and competitive records.

Do not delete the previous season's ratings when a new season starts.

Create new season-scoped rating records and preserve historical results. If a soft reset is used, calculate the new rating from the previous rating according to documented rules.

Season rewards must be issued once per eligible player. Track reward grants with a unique reference so retrying a season-finalization job cannot duplicate rewards.

The season's final leaderboard must be based on validated data at the defined cutoff.

---

## 13. Friends, Presence, Parties, and Squads

### 13.1 Friendships

Use a `friendships` table with a stable pair of user identifiers and a relationship status.

Possible fields:

- `id`
- `requesterId`
- `addresseeId`
- `status`
- `createdAt`
- `updatedAt`

Possible statuses include pending, accepted, and rejected, depending on the final social design.

Normalize the user pair or otherwise enforce uniqueness so the same friendship cannot be created in both directions as separate accepted relationships.

### 13.2 Presence

Online presence is temporary.

Keep current connection state in the real-time server or the selected framework's presence mechanism. Do not write every online/offline transition or heartbeat to PostgreSQL.

Persist last-seen information only if the product needs it. Treat it as an approximate activity timestamp, not proof that a player is currently online.

If multiple server instances are introduced, presence coordination will need a deliberate cross-instance strategy.

### 13.3 Parties and squads

Distinguish between a temporary party and a persistent squad or clan.

A party exists to group players for matchmaking or a private session. Its live membership can remain in server memory.

A persistent squad, if implemented as a durable social feature, needs database records such as:

- `squads`
- `squad_members`
- `squad_invitations`

Membership records should define the member's role and join date. Enforce one membership per player if that is the chosen product rule.

Do not create persistent squad tables simply to implement temporary match parties.

### 13.4 Invitations

Invitations should have a clear recipient, sender, target, status, and expiry.

Temporary party and match invitations can live in memory if losing them on a server restart is acceptable.

Persist them if the product requires offline delivery, history, or reliable cross-instance handling. This is a product and reliability decision, not a reason to persist every transient event by default.

### 13.5 Blocks

Use a player-block relationship to enforce social restrictions.

A block must be checked by the server when processing relevant interactions. The client may cache block state for responsiveness, but the cache must not replace the authoritative restriction.

---

## 14. Chat Storage and Safety

Chat has different storage needs from gameplay state.

### 14.1 Live delivery

The real-time server delivers chat to authorized recipients in the appropriate lobby, squad, or team channel.

The server validates channel membership, message length, rate limits, mute status, and applicable restrictions before broadcasting.

### 14.2 Persistence policy

Do not persist every chat message indefinitely by default.

Use a defined policy:

- Live gameplay chat is delivered in real time.
- Persistent lobby or direct-message history is supported only if the product requires it.
- Match chat may be retained for a limited period if needed for moderation or reports.
- Relevant reported messages may need contextual evidence, subject to a documented retention policy.
- Chat history must not be publicly accessible merely because the participants played the same match.

If persistence is enabled, a `chat_messages` table may contain:

- `id`
- `channelType`
- `channelId`
- `senderId`
- `content`
- `createdAt`
- `deletedAt` — optional moderation marker.
- `moderationStatus` — optional.

Do not store message content in a general application log. Restrict access to any retained chat records.

### 14.3 Retention

Define retention periods before enabling persistent chat at scale.

The policy should specify:

- Which channels are stored.
- How long ordinary messages are retained.
- Whether reported messages are retained differently.
- Who can access retained messages.
- How deletion, moderation, and applicable data requests work.

The exact retention period remains an open policy decision. Avoid promising private or ephemeral chat unless the implementation and retention rules actually support that promise.

---

## 15. Reports, Moderation, and Administration

### 15.1 Player reports

Use a `player_reports` table for durable reports.

Potential fields:

- `id`
- `reporterId`
- `reportedUserId`
- `matchId` — optional.
- `category`
- `description`
- `evidenceReference` — optional.
- `status`
- `assignedModeratorId` — optional.
- `createdAt`
- `resolvedAt` — optional.

Use a controlled category list and bounded descriptions.

The reporter's identity and report contents must not be exposed to the reported player through ordinary player-facing endpoints.

### 15.2 Moderation actions

Use a `moderation_actions` table for warnings, chat restrictions, temporary bans, permanent bans, and reversals.

Potential fields:

- `id`
- `targetUserId`
- `actorUserId`
- `actionType`
- `reason`
- `startsAt`
- `endsAt` — nullable.
- `relatedReportId` — optional.
- `createdAt`

The server must enforce active restrictions. An account's UI state is not the source of truth for whether it is banned or muted.

For consequential actions, preserve the action history rather than simply overwriting a player's previous status.

### 15.3 Administrative audit log

Use an `admin_audit_logs` table for sensitive administrative operations.

Potential fields:

- `id`
- `actorUserId`
- `actionType`
- `targetType`
- `targetId`
- `reason`
- `metadata`
- `createdAt`

Store only the metadata needed to explain the action. Do not put passwords, session tokens, or unnecessary private information into audit records.

Administrative permissions must be enforced server-side. Audit logging should capture consequential changes, including role changes and moderation reversals.

---

## 16. Database Schema and Constraints

Use Drizzle ORM to define the PostgreSQL schema and migrations.

The schema should establish:

- Primary keys.
- Foreign keys.
- Unique constraints.
- Check constraints where appropriate.
- Indexes for common access patterns.
- Explicit nullability.
- Consistent timestamp conventions.
- Transaction boundaries for related updates.

### 16.1 Identifier strategy

Use UUIDs or another consistent identifier strategy for durable entities.

Do not mix unrelated identifier formats without a reason. External asset keys and slugs can remain human-readable while internal relational identifiers remain stable.

### 16.2 Important uniqueness constraints

Examples include:

- Unique normalized usernames.
- Unique authentication provider identities.
- One player profile per user.
- One inventory ownership record per player and unique item.
- One currency account per player and currency.
- Unique idempotency keys within their relevant operation scope.
- One ranked rating record per player, mode, and season.
- One participant record per player and match.
- Unique player-pair friendships.
- One squad membership per player, if only one persistent squad is permitted.

These constraints prevent application retries and concurrent requests from silently corrupting data.

### 16.3 Transactions

Use database transactions for operations that must succeed or fail together.

Examples:

- Cosmetic purchase and ownership grant.
- Currency reward and transaction ledger entry.
- Match result finalization and statistics updates.
- Ranked rating updates.
- Season reward grants.
- Account initialization and starter-item grants.

For operations involving multiple players, such as team rating updates, define the transaction boundaries carefully so players cannot end up with partially updated results.

### 16.4 Drizzle migrations

Treat migrations as versioned repository artifacts.

- Keep schema definitions in a clearly owned database package or server module, consistent with the existing monorepo.
- Generate and review migrations before applying them.
- Never use production schema push operations as an unreviewed substitute for migration management.
- Test migrations against a development database.
- Preserve existing data when changing schema.
- Document destructive migrations and any necessary backfills.

The exact directory layout must match the current repository after inspection.

---

## 17. Indexing and Query Patterns

Add indexes based on real access patterns rather than indexing every column.

Likely candidates include:

- User lookup by normalized username.
- Inventory lookup by user.
- Currency accounts by user and currency.
- Match history by participant and match end time.
- Match participants by match identifier.
- Ranked ratings by mode, season, and descending rating.
- Seasonal statistics by season and metric.
- Friendships by participant and status.
- Squad memberships by squad and user.
- Chat messages by channel and creation time, if persistent chat is enabled.
- Open reports by status and creation time.
- Active moderation restrictions by target user and expiry.

Leaderboard indexes must support the intended sorting and filtering patterns. Validate query plans as data volume grows.

Use pagination for large history lists. Prefer stable cursor-based pagination where offsets become expensive or results change frequently.

Do not load every player's entire inventory, every historical match, or every leaderboard entry into memory for one request.

---

## 18. Caching and Derived Data

Neon is the persistent source of truth, but not every read must trigger an expensive database query.

Potential derived or cached data includes:

- Public player profile summaries.
- Current leaderboard pages.
- Static game catalog metadata.
- Asset manifests.
- Frequently used match configuration.

Static assets should be cached by the browser and the delivery layer using suitable cache headers and versioned URLs.

Live player state should be managed by the real-time server, not cached in PostgreSQL as if it were a static lookup.

Any cache of ownership, currency, permissions, or restrictions must have a clear invalidation strategy. A stale cache must not allow a player to spend unavailable currency, equip an unowned item, or bypass an active ban.

Start with simple database queries and in-process caching where appropriate. Add a separate cache service only when measurement demonstrates a need.

---

## 19. Persistence Boundaries

The following rules define what should be written to durable storage.

| Data | Primary location | Persistence rule |
|---|---|---|
| Account and profile | Neon | Durable |
| Owned items | Neon | Durable |
| Currency balances and ledger | Neon | Durable and transactional |
| XP and progression | Neon | Durable |
| Ranked rating and history | Neon | Durable |
| Completed match results | Neon | Durable |
| Live positions and health | Server memory | Temporary match state |
| Current ammunition and reload timer | Server memory | Temporary match state |
| Active room membership | Real-time server | Temporary |
| Live presence | Real-time server | Temporary, unless a limited last-seen field is needed |
| Chat messages | Real-time server | Delivered live; persist only under defined policy |
| Character and weapon models | Render Static Site | Static files |
| Maps, textures, and audio | Render Static Site | Static files |
| Asset metadata | Neon or versioned catalog configuration | Durable metadata |
| Moderation reports | Neon | Durable |
| Administrative audit records | Neon | Durable |
| Browser camera and interpolation state | Browser memory | Temporary |
| Client input state | Browser memory | Temporary |

The authoritative owner for each type of data must remain unambiguous.

---

## 20. Reliability and Recovery

Persistent operations must account for retries, process restarts, and partial failures.

- Give matches stable identifiers before result finalization.
- Make reward and result processing idempotent.
- Use transactions for related financial, progression, and ranking updates.
- Preserve match history even if a player's aggregate statistics need rebuilding.
- Record enough processing state to recover interrupted result finalization.
- Handle Neon connectivity failures without pretending a persistent write succeeded.
- Do not tell the client that currency, ownership, or ranked rating has been saved until the server has confirmation.

If Neon is temporarily unavailable during a live match, define which gameplay actions may continue and which persistent actions must be delayed or rejected. Never grant uncommitted currency or ownership as if it were durable.

A full server-process restart may end active matches unless match recovery is explicitly implemented. Communicate this behavior and finalize results safely wherever possible.

---

## 21. Privacy and Data Lifecycle

Store only data that has a clear gameplay, operational, safety, or product purpose.

Define lifecycle rules for:

- Account deletion and anonymization.
- Profile visibility.
- Match-history retention.
- Chat retention.
- Reports and moderation evidence.
- Administrative audit records.
- Inactive accounts.
- Season history.

Some records may need to be retained for security, competitive integrity, or dispute handling, subject to the applicable policy and legal requirements.

Avoid deleting or anonymizing records in a way that corrupts match history, financial ledgers, or audit trails. Decide how historical references should behave when an account is deleted.

---

## 22. Recommended Implementation Order

### Phase 1 — Database foundation

- Configure Neon connectivity.
- Establish Drizzle schema and migrations.
- Implement users and player profiles.
- Add catalog records for initial characters, weapons, cosmetics, maps, and modes.
- Define asset manifest references.
- Add starter inventory and equipment validation.

### Phase 2 — Core match persistence

- Persist completed matches and participants.
- Store match statistics.
- Make result processing idempotent.
- Add profile match history.
- Implement aggregate statistics.

### Phase 3 — Progression and economy

- Add XP and progression.
- Add currency accounts and transaction ledger.
- Implement inventory grants and cosmetic purchases.
- Add achievements and reward rules.
- Test transaction rollback and duplicate requests.

### Phase 4 — Competitive systems

- Add seasons.
- Implement ranked rating and rating history.
- Build indexed leaderboard queries.
- Add eligibility, tie-breaking, and season-finalization rules.
- Validate rating calculations with deterministic tests.

### Phase 5 — Social and safety

- Add friendships, blocks, and persistent squads if required.
- Implement the selected chat persistence policy.
- Add reports, moderation actions, bans, and audit logs.
- Verify role enforcement and data access restrictions.

### Phase 6 — Scale and operations

- Measure database query performance.
- Add indexes and caching where justified.
- Review migration and recovery procedures.
- Test concurrent purchases and reward processing.
- Verify static asset caching and versioning.
- Review retention, deletion, and privacy behavior.

---

## 23. Definition of Done

The data architecture is ready to guide implementation when:

- Neon PostgreSQL is the documented persistent source of truth.
- Drizzle owns schema definitions and versioned migrations.
- Live game state is separated from durable account and match data.
- Asset binaries are served through the static asset host rather than stored as database blobs.
- Inventory ownership and equipment are separate concepts.
- Currency and rewards are transactional and idempotent.
- Progression, ranked rating, and leaderboard statistics have defined sources.
- Match results can be safely processed without duplicate rewards or ratings.
- Social, chat, reporting, and moderation data have explicit persistence policies.
- Database constraints protect critical relationships.
- Query patterns, indexes, caching, and recovery behavior are documented.
- The final schema has been reconciled with the actual monorepo and authentication implementation.

**Core rule:** Neon remembers what a player owns and has achieved. The real-time server decides what happens in the current match. Static hosting delivers the assets that make the world visible.

