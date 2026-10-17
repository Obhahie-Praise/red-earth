# Red Earth — Product Vision

## 1. Overview

Red Earth is a browser-first multiplayer first-person shooter inspired by Nigeria, Edo heritage and the environments of Benin City.

The game combines tactical combat with a distinctive, stylized-realistic visual identity built around red earth, Nigerian environments and cultural details. It aims to deliver an engaging multiplayer experience directly through the browser, without requiring players to download a large standalone game.

Red Earth is not intended to be another generic shooter with a Nigerian map added to it. Its setting, visual identity, environments and community should contribute to a recognizable identity of its own.

## 2. Vision

Build a recognizable, accessible multiplayer FPS that brings Nigerian-inspired environments and identity into an engaging online gaming experience.

The game should be fun to play, enjoyable to explore and social even when players are not actively fighting.

## 3. Target Audience

Red Earth is intended for:

- Players who enjoy competitive FPS games and tactical combat.
- Players looking for browser-based games that do not require a large download.
- Nigerian and African players interested in seeing familiar environments and cultural influences represented in games.
- Friends and gaming communities who want to play together, customize their characters and compete.

The game should also accommodate players using modest computers and internet connections wherever technically feasible.

## 4. Product Principles

### 4.1 Browser-first accessibility

Players should be able to access the game through a supported web browser. Avoid requiring a large standalone game installation.

The experience should prioritize reasonable loading times, efficient asset delivery and performance-conscious rendering.

### 4.2 Multiplayer from the beginning

Online multiplayer is a core part of Red Earth, not an optional feature to be added after building a single-player game.

Networking, authoritative game-state handling and multiplayer testing must inform the architecture from the start.

### 4.3 A distinct visual identity

Red Earth should have a recognizable visual language inspired by red earth, Edo environments and Nigerian cultural influences.

The intended direction is stylized realism with painterly qualities, atmospheric environments and carefully chosen cultural details.

Authenticity should come from considered environmental design rather than superficial decoration.

### 4.4 Social play matters

The experience should extend beyond individual matches.

Players should be able to meet friends, form squads, send invitations, communicate through text chat, customize their characters, inspect weapon cosmetics and express themselves through emotes.

### 4.5 Competitive integrity and safety

The server must remain authoritative over important competitive outcomes. Client-side actions must not be trusted as proof of legitimate results.

Players should have access to reporting and blocking tools, while authorized administrators need secure moderation capabilities.

### 4.6 Performance before unnecessary complexity

Visual quality should not come at the cost of making the game unusable on modest hardware.

Use efficient assets, appropriate levels of detail, sensible scene management and progressive loading. Features should justify their implementation and ongoing operational costs.

## 5. Experience Principles

Red Earth should feel like a game rather than a SaaS dashboard.

Its interface should use a consistent dark theme, strong typography, restrained corner rounding and purposeful animation. The 3D environment should provide much of the visual richness.

The lobby should feel alive. Players should see their characters, inspect equipment, interact with friends and prepare for matches without navigating a collection of disconnected settings screens.

Loading states should communicate what is happening instead of leaving players staring at a frozen screen. Progress percentages must represent measurable progress; otherwise, use honest indeterminate feedback.

The interface should be responsive, but full FPS gameplay should prioritize desktop keyboard-and-mouse controls in V1. Mobile browsing and interface support must not be confused with a commitment to full mobile combat.

## 6. V1 Direction

The first version is intended to establish a playable, online multiplayer foundation with a coherent visual identity and social experience.

Its planned direction includes:

- Browser-based 3D gameplay.
- Multiplayer matches and multiple game modes.
- Distinct maps and environments.
- Character and weapon customization.
- Squads, invitations and text chat.
- A social lobby with emotes.
- Player reporting and essential administration tools.
- Responsive non-gameplay interfaces and a desktop-first combat experience.

Detailed gameplay requirements, exact content quantities and feature priorities will be defined in the game design, feature specification and roadmap documents.

These are product intentions, not claims that the features have already been implemented.

## 7. Technical Direction

The current intended foundation is:

- **Monorepo:** pnpm and Turborepo.
- **Web application:** Next.js, React and TypeScript.
- **3D rendering:** Three.js.
- **Game server:** NestJS.
- **Real-time communication:** Socket.IO.
- **Shared contracts:** shared packages where useful and actually configured.

The repository's installed dependencies, existing structure and working configuration must be inspected before implementation decisions are made. Documentation must reflect the actual repository state rather than assuming every planned component is already configured.

## 8. Non-goals and Constraints

Red Earth should not attempt to implement every possible feature at launch.

The following constraints guide V1:

- No voice chat in V1; text communication is the priority.
- No requirement for a standalone game download.
- No assumption that full mobile FPS controls must ship with the first version.
- No unnecessary paid infrastructure before its value and affordability are established.
- No trust in client-reported competitive results without server validation.
- No presentation of planned features as completed features.

## 9. Definition of Success

Red Earth is moving in the right direction when players can access the game reliably, join and complete multiplayer matches, invite friends, communicate safely and recognize the game's distinctive identity.

Success should ultimately be evaluated using real player activity, match completion, connection reliability, repeat play, community growth and feedback.

The initial priority is to establish a working and enjoyable foundation before expanding the game with additional content and systems.