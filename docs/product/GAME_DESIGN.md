# Red Earth — Game Design Document

**Status:** Draft  
**Scope:** V1 gameplay and player experience  
**Product:** Red Earth, a browser-first multiplayer FPS

## 1. Design Goals

Red Earth should deliver tactical, responsive multiplayer combat within a distinctive world inspired by Nigeria and Edo heritage.

The game should balance four priorities:

1. **Combat:** Shooting, movement, positioning and teamwork should feel responsive and rewarding.
2. **Identity:** Maps, characters, weapons and environments should contribute to a recognizable Red Earth aesthetic.
3. **Social play:** Players should have meaningful reasons to interact before, during and after matches.
4. **Accessibility:** The browser-based experience should remain practical for players using modest computers and internet connections.

## 2. Core Player Loop

The intended player journey is:

1. Open Red Earth and load into the game.
2. Enter the lobby and see their character, equipment and available social features.
3. Invite friends or join a squad.
4. Select a game mode and prepare for a match.
5. Play a multiplayer match.
6. Receive match results, rewards and progression updates.
7. Customize equipment, interact with friends and play again.

The lobby and progression systems should encourage repeat play without forcing players through unnecessary menus.

## 3. Camera and Combat Perspective

Red Earth is a first-person shooter.

Players should experience combat from their character's perspective, with the weapon and relevant equipment visible on screen.

Character models remain important for the lobby, customization, emotes and other situations where third-person presentation is appropriate.

### Proposed combat principles

- Mouse-based aiming and keyboard movement on desktop.
- Responsive firing, reloading and weapon switching.
- Clear feedback for hits, damage and eliminations.
- Meaningful differences between weapon classes.
- Movement and weapon handling that reward positioning rather than unrestricted spraying.
- Server-validated combat outcomes.

Exact movement speeds, recoil patterns, damage values, fire rates and time-to-kill targets remain to be determined through prototyping and playtesting.

## 4. Game Modes

The intended long-term V1 scope includes at least three game modes. The following are proposed candidates, not yet finalized specifications.

### 4.1 Battle Royale

Players or squads enter a larger map, search for equipment and attempt to survive until the match ends.

Potential mechanics include:

- A large playable environment assembled from suitable map areas.
- Loot and equipment distribution.
- A shrinking playable zone.
- Solo and squad variants, if supported by the final design.
- A victory condition based on the last surviving player or squad.

This mode has substantial technical requirements, including larger environments, more simultaneous players, matchmaking and authoritative game-state management. Its implementation scope must be evaluated against the capabilities of the initial server architecture.

### 4.2 Team Deathmatch

Two teams compete to reach a target score or lead when the match timer expires.

This is a strong candidate for the first playable mode because it can support smaller maps, straightforward objectives and relatively quick testing of combat fundamentals.

Proposed mechanics include team spawns, a score limit, a match timer and a results screen.

### 4.3 Free-for-All

Players compete individually for the highest score or the most eliminations within a time limit.

This mode provides an alternative to team-based combat and can help test individual weapon balance and player movement.

### Additional modes

Other possibilities include objective capture, elimination rounds and private duel matches. These should be evaluated after the initial combat and networking foundations work reliably.

## 5. Maps and Environments

The intended content goal is at least five maps, with suitable areas potentially combined or adapted for a larger battle royale environment.

Maps should draw inspiration from Edo heritage and Nigerian environments while remaining readable and balanced for FPS combat.

Potential environmental themes include:

- Urban streets and neighbourhoods inspired by Benin City.
- Traditional architecture and historically informed surroundings.
- Markets, courtyards and public spaces.
- Rural terrain and red-earth landscapes.
- Industrial or contemporary environments that fit the game's setting.

These are creative directions, not approved map names or confirmed locations.

### Map design principles

Each map should provide:

- Clear routes and recognizable landmarks.
- A balance of open sightlines and close-quarters spaces.
- Cover that supports tactical movement.
- Spawn locations designed to reduce immediate spawn trapping.
- Readable geometry for competitive play.
- Appropriate lighting and visibility.
- Performance-conscious environmental assets.

Map design must consider both combat and browser performance. Decorative detail should not compromise visibility, collision accuracy or frame rate.

### Map validation

Before a map is considered ready, it should be tested for:

- Collision and traversal issues.
- Spawn safety.
- Line-of-sight imbalances.
- Performance on the project's minimum supported hardware.
- Network synchronization issues.
- Suitability for its intended game mode.

## 6. Weapons and Equipment

The intended weapon roster includes:

| Category | Planned quantity |
|---|---:|
| Assault rifles | 3 |
| Submachine guns | 3 |
| Shotguns | 2 |
| Grenades | Planned |
| Flashbangs | Planned |

These quantities are intended content targets, not implemented inventory.

### Weapon design principles

Weapons should have distinct roles instead of functioning as interchangeable models with different appearances.

- **Assault rifles:** general-purpose combat at medium ranges.
- **SMGs:** close-range mobility and rapid fire.
- **Shotguns:** powerful close-range engagements with limited effective range.
- **Grenades:** area denial or explosive damage, depending on the finalized design.
- **Flashbangs:** temporary disruption of an opponent's ability to fight effectively.

Exact weapon statistics must be established through playtesting.

### Equipment and combat validation

The server must validate ammunition, firing cadence, reload state, damage, weapon switching and equipment use.

Client-side animations can provide immediate visual feedback, but they must not independently determine authoritative damage or match outcomes.

## 7. Characters and Customization

The planned initial character roster contains at least five characters.

The design should support a default appearance and complimentary starter cosmetic options, subject to the final asset scope.

Customization may include:

- Character skins.
- Weapon skins.
- Equipped weapons and loadouts.
- Cosmetic emotes.
- Other non-gameplay visual items introduced later.

Cosmetics should not provide direct competitive advantages.

### Character presentation

Characters should be recognizable in the lobby and visually distinguishable during combat where the camera and game rules allow.

Cosmetic designs must maintain gameplay readability. Extremely bright effects, oversized accessories or visual clutter should not make opponents difficult to identify.

## 8. Weapon Skins and Cosmetic Progression

Players should be able to inspect their weapons and see their equipped skins in the lobby.

The initial customization experience should prioritize:

- Viewing owned cosmetics.
- Previewing a selected cosmetic.
- Equipping and unequipping supported items.
- Seeing the currently equipped appearance.
- Returning to the lobby without losing changes.

A future progression system may introduce unlockable skins, rewards, challenges and cosmetic collections.

The exact economy, unlock rules, rarity tiers and monetization model are not yet finalized. These systems must not be represented as implemented until they exist.

## 9. Lobby and Social Experience

The lobby is a social space and the main staging area before a match.

It should combine an engaging 3D character presentation with convenient access to social and gameplay actions.

### 9.1 Lobby presentation

The proposed direction is a cinematic 3D lobby with practical social controls.

Players should be able to:

- See their current character.
- Rotate or inspect their character where supported.
- Preview their equipped weapon and skins.
- Open character customization and loadout screens.
- Invite friends.
- View friends' online presence.
- Form or join a squad.
- Choose an available game mode.
- Start matchmaking or enter a supported private match.
- Use emotes.

The environment should have atmosphere and subtle animation without making menus slow or obstructing access to gameplay.

### 9.2 Squads and invitations

Squads allow friends to prepare and enter matches together.

The squad system should eventually support:

- Sending and accepting invitations.
- Joining or leaving a squad.
- Displaying squad membership and player presence.
- Showing the squad leader.
- Allowing authorized members to leave or manage the party.
- Communicating squad status to matchmaking.

The exact maximum squad size and invitation permissions remain open decisions.

### 9.3 One-versus-one matches

Players should be able to challenge another player to a 1v1.

The proposed flow is:

1. Select or invite a player.
2. Send a duel challenge.
3. Allow the recipient to accept or decline.
4. Select a supported duel map and rules, where available.
5. Start a private match.
6. Display the result when the duel ends.

Private duels should use the same server-authoritative combat rules as other modes.

Ranked duels, spectating and custom rule editors are not required unless explicitly added to the V1 scope.

## 10. Text Chat

Voice chat is excluded from V1. Text chat is the planned communication system.

The intended channels are:

- **Lobby chat:** communication among players sharing a lobby.
- **Squad chat:** communication among squad members.
- **Team chat:** communication with teammates during a match, where supported.

The interface should clearly distinguish channels and show message delivery and system notifications where appropriate.

### Chat safety

The chat system should support:

- Muting and blocking players.
- Reporting inappropriate messages or player behaviour.
- Rate limiting and spam prevention.
- Server-side message validation.
- Appropriate moderation controls.
- Clear handling of disconnected players and failed message delivery.

Chat messages must not be treated as trusted input. User-generated content should be safely rendered and validated.

Voice chat may be considered in a later release, but the V1 implementation should not depend on it.

## 11. Emotes

Emotes are planned as a small, expressive cosmetic feature.

The initial selection should be limited enough to produce and test without delaying core gameplay.

Potential starter emotes include:

- Wave or greeting.
- Celebration.
- Laughter.
- Dance.
- A distinctive Red Earth greeting or gesture.

Players should be able to trigger emotes through a simple selector or quick-access wheel.

### Emote requirements

- Emotes should use reusable character animations wherever possible.
- Only supported emotes should be triggerable.
- Repeated activation should be limited to prevent spam.
- Emotes must not interrupt critical gameplay controls unexpectedly.
- Emotes should not grant competitive advantages.
- Animation loading should fail gracefully if an asset is unavailable.

The final starter emote list, animation style and match restrictions remain open for review.

## 12. Match Flow and Results

A match should have clear transitions between preparation, active play and completion.

The intended flow is:

1. Match preparation or countdown.
2. Player spawning.
3. Active gameplay.
4. Match-end condition.
5. Results screen.
6. Return to the lobby or proceed to another match.

The results screen should display relevant information, such as team outcome, individual performance and match duration, depending on the selected mode.

Only validated server results should determine official scores, rewards and competitive progression.

## 13. Ranking and Progression

A ranking system is part of the intended broader game experience.

Its design should distinguish competitive rank from general account progression and cosmetic ownership.

Potential progression elements include:

- Match history.
- Player statistics.
- Competitive ranking.
- Experience or account levels.
- Cosmetic unlocks.
- Leaderboards.

The initial ranking algorithm, seasonal structure, reward rules and anti-abuse measures are not yet finalized.

Ranked play should not launch until match results, disconnections, suspicious activity and score validation can be handled reliably.

## 14. In-Game Currency and Rewards

An in-game currency and cosmetic reward system are planned design goals.

The currency should initially serve a clearly defined purpose, such as unlocking cosmetics or other non-competitive items.

Before implementation, the following must be specified:

- How currency is earned.
- What it can purchase.
- Whether balances persist across sessions.
- How transactions are validated.
- How duplicate requests and abuse are prevented.

The economy must not allow client-side manipulation of balances or rewards.

The initial release should prioritize a reliable and understandable progression model over a complex economy.

## 15. Player Safety and Moderation

Players should have practical ways to manage unwanted interactions and report misconduct.

The game should provide reporting options for relevant categories, including:

- Cheating or exploiting.
- Harassment or abusive chat.
- Spam.
- Inappropriate player names.
- Other reportable conduct defined by the moderation policy.

Where appropriate, reports should retain relevant contextual identifiers, such as the reported player, match and message reference.

Players should not be shown another player's private account details or the identity of a reporter merely because a report was submitted.

Authorized administrators should be able to review reports and apply proportionate moderation actions.

## 16. Mobile and Input Strategy

The interface should be responsive across supported screen sizes.

V1 combat should prioritize desktop devices using keyboard and mouse.

Responsive non-gameplay interfaces may include account access, profile viewing, settings and supported social features.

Full mobile combat requires separate decisions about touch controls, aiming assistance, performance budgets, UI scaling and fairness between input methods. It should not be assumed to be complete merely because the website adapts to a small screen.

## 17. Loading and Performance

Loading is part of the gameplay experience.

The game should provide appropriate feedback for:

- Initial startup.
- Asset loading.
- Map preparation.
- Multiplayer connection.
- Matchmaking.
- Match initialization.
- Recoverable connection failures.

Use measurable progress when available and honest indeterminate feedback otherwise.

Assets should be loaded progressively where practical, and the implementation must manage Three.js resources, cached assets, texture sizes and memory usage carefully.

Performance should be evaluated on modest hardware rather than only on high-end development machines.

## 18. Scope and Open Decisions

The following require further definition before they can be treated as finalized implementation requirements:

- Exact map names, layouts and game-mode assignments.
- Initial player counts for each mode.
- Squad size and party permissions.
- Battle royale zone mechanics and loot rules.
- Weapon statistics and damage model.
- Respawn rules and match durations.
- Ranked-play formula and reward structure.
- Cosmetic unlocks and currency economy.
- Exact starter emotes and animation assets.
- The precise boundary between mobile social features and gameplay.
- Minimum supported browser, resolution and hardware requirements.

These decisions should be recorded in the project decision log and reflected in the roadmap when settled.

## 19. Design Completion Criteria

The game design is ready to guide implementation when:

- Core combat rules are defined well enough to prototype.
- Each initial game mode has clear win and end conditions.
- The first playable map has a defined layout and gameplay purpose.
- Lobby, squad and chat interactions are specified.
- The V1 cosmetic and emote scope is agreed.
- Server-authoritative responsibilities are documented.
- Performance and minimum-device expectations are measurable.

This document describes the intended game design. It does not certify that any feature is implemented or working.