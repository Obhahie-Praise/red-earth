# Red Earth — Security Architecture

**Path:** `docs/architecture/SECURITY.md`  
**Status:** Draft — implementation details must be verified against the existing repository  
**Scope:** Browser client security, multiplayer integrity, authentication, moderation, administrative access, and data protection.

## 1. Purpose

Red Earth is a competitive multiplayer FPS with social features. Security must protect players, preserve fair matches, and prevent unauthorized access to accounts, game state, and administrative tools.

The browser is an untrusted environment. Players can inspect client code, modify requests, replay messages, and run modified clients.

Our goal is not to promise perfect protection against cheating. It is to establish sensible security boundaries from the beginning and make abuse detectable and manageable.

## 2. Core Security Principles

1. **The server is authoritative.** Clients cannot decide damage, eliminations, scores, match results, or rewards.
2. **Validate every untrusted input.** TypeScript types do not replace runtime validation.
3. **Enforce permissions on the server.** Hiding a button is not access control.
4. **Collect only necessary data.** Do not retain sensitive information without a defined purpose.
5. **Make moderation accountable.** Privileged actions must be authorized and auditable.
6. **Fail safely.** Invalid commands must not corrupt game state or expose private data.
7. **Keep security practical.** Prioritize real risks instead of adding unnecessary infrastructure before it is needed.

## 3. Threat Model

The system should account for the following threats.

| Threat | Required protection |
|---|---|
| Modified game client | Server-side validation of gameplay actions |
| Forged damage or score | Server-authoritative combat and scoring |
| Invalid or oversized messages | Runtime schemas, payload limits, and rejection |
| Spam and flooding | Per-user and per-connection rate limits |
| Account impersonation | Verified identity and secure session handling |
| Unauthorized room access | Server-checked membership and permissions |
| Chat harassment | Mute, block, reporting, and moderation |
| Fake administrator actions | Server-enforced role checks |
| Stolen session credentials | Secure cookies or equivalent session protection |
| Database exposure | Least privilege, secret management, and restricted access |
| Cheating reports abused maliciously | Review procedures and auditable moderation |
| Service disruption | Connection limits, resource limits, and monitoring |

This threat model should evolve as Red Earth gains real users and new features.

## 4. Authentication and Sessions

The authentication system must establish who a player is before granting access to protected account features.

Requirements:

- Use a maintained authentication solution appropriate to the selected backend.
- Validate sessions on the server.
- Use secure, HTTP-only cookies when the authentication architecture supports cookie-based sessions.
- Configure `Secure` cookies in production and an appropriate `SameSite` policy.
- Protect state-changing HTTP requests against cross-site request forgery where applicable.
- Validate allowed origins and configure CORS explicitly.
- Avoid storing long-lived authentication tokens in browser local storage when a safer supported session approach is available.
- Expire and revoke sessions appropriately.
- Never log passwords, session tokens, or complete authentication credentials.

For real-time connections, establish and validate the player's identity during connection or room entry. A client-supplied account ID alone is not proof of identity.

If a session expires or is revoked, the server must prevent further privileged actions and handle the connection according to the session policy.

**Implementation rule:** Inspect the repository's current authentication setup before selecting a new library or changing session behavior.

## 5. Multiplayer Integrity and Anti-Cheat

### 5.1 Server-authoritative gameplay

The server must validate movement, weapon state, firing cadence, damage, eliminations, scores, match completion, and rewards.

Clients may send input commands and report local observations. They must not be trusted to supply final competitive outcomes.

### 5.2 Movement validation

The server should reject or correct movement that violates the current movement model, including impossible speeds, invalid transitions, and movement beyond allowed limits.

Validation must account for legitimate latency and the chosen movement simulation. Avoid simplistic checks that punish players for ordinary network instability.

### 5.3 Weapon validation

The server must enforce:

- Equipped weapon and ownership.
- Ammunition and reload state.
- Fire rate and cooldown.
- Weapon-specific behavior.
- Valid player and match state.
- Server-side hit and damage rules.

Do not accept a client-provided damage value as authoritative.

### 5.4 Abuse detection

Record useful signals such as repeated invalid commands, impossible movement, excessive message frequency, and repeated attempts to access unauthorized rooms.

A suspicious signal is not automatically proof of cheating. Use appropriate thresholds and investigation before applying severe penalties.

Automated enforcement should begin with conservative measures, such as rejecting invalid commands and limiting abusive connections. Permanent bans should not depend on a single unreliable signal.

### 5.5 Limits of anti-cheat

Server authority reduces opportunities for cheating but cannot prevent every form of exploitation. Information already sent to a browser may be extracted, and compromised accounts may still be abused.

Do not promise that Red Earth is cheat-proof.

## 6. Network Input Validation

Every HTTP request and real-time message must be treated as untrusted.

The server should validate:

- Required fields and data types.
- String lengths and permitted characters.
- Numeric ranges and finite values.
- Array sizes and nested payload depth.
- Message size and frequency.
- Current player state and permissions.
- Room membership and match context.

Reject malformed messages before they reach gameplay logic.

Use shared TypeScript contracts where practical, alongside runtime validation at the server boundary. Keep error messages useful without exposing secrets or internal implementation details.

Apply rate limits according to action type. Sending movement input has different requirements from sending a chat message or requesting an invitation.

## 7. Room and Resource Protection

The server must not allow players to join arbitrary rooms simply by supplying a room identifier.

Validate room existence, visibility, capacity, match state, invitations, and membership requirements.

Set sensible limits on:

- Simultaneous connections per account or session.
- Room membership.
- Message payload sizes.
- Pending invitations.
- Chat frequency.
- Reconnection attempts.
- Expensive operations.

Avoid unlimited in-memory state, unbounded queues, and unbounded collections of disconnected players.

When a connection closes, release temporary resources and remove subscriptions that are no longer needed.

## 8. Chat and Social Safety

Red Earth V1 includes text chat. Voice chat is not included in V1.

### 8.1 Chat rules

The server should:

- Validate and limit message length.
- Apply message rate limits.
- Enforce lobby, squad, and team-channel membership.
- Respect mute and block restrictions.
- Prevent unauthorized users from sending messages as another player.
- Restrict spam and repeated messages.
- Provide a path for reviewing reported messages where practical.

Do not render user-generated text as trusted HTML. Escape or safely render chat content to reduce cross-site scripting risk.

Any automated content filtering should be introduced with a clear policy and tested for false positives.

### 8.2 Mute and block

Players should be able to mute or block other players according to the product's social rules.

The system must clearly define whether muting affects only local display, whether blocking also affects direct interactions, and how those settings persist across sessions.

Do not claim a player is blocked globally if the implementation only hides their messages on one client.

### 8.3 Player reports

Players should be able to report relevant misconduct, including:

- Suspected cheating.
- Harassment or abusive chat.
- Spam.
- Inappropriate display names.
- Other defined rule violations.

A report should contain only the information necessary for review, such as the reported player, category, time, and relevant contextual evidence when available.

Do not promise that every action can be reconstructed if the system does not retain the necessary evidence.

Protect report confidentiality and restrict access to authorized reviewers.

## 9. Moderation and Administrative Access

Administrative functionality must be protected independently of the client UI.

### 9.1 Roles and permissions

Use explicit roles or permissions for administrative actions. The exact role model can remain simple initially.

For example:

- **Player:** ordinary gameplay and personal account actions.
- **Moderator:** authorized player reports and defined moderation actions.
- **Administrator:** broader account, moderation, and operational management.

These are proposed role categories; final roles must match the implemented product.

Every privileged operation must check authorization on the server. Never grant administrative access because a client sends `isAdmin: true`, changes a profile field, or opens a hidden route.

### 9.2 Moderation actions

Depending on the final moderation policy, authorized staff may be able to:

- Review reports.
- Issue warnings.
- Apply temporary restrictions.
- Restrict chat.
- Temporarily suspend accounts.
- Permanently ban accounts.
- Reverse or expire restrictions.
- Review prior moderation actions.

Require a reason for consequential actions. Apply stronger confirmation to destructive or permanent actions.

### 9.3 Audit log

Record consequential administrative actions, including the actor, target, action, timestamp, reason, and outcome where appropriate.

Restrict access to audit records and avoid placing passwords, session tokens, or unnecessary private information in them.

An administrator should not be able to silently rewrite the history of their own moderation actions through ordinary application functionality.

## 10. Account and Game Data Protection

Classify data according to sensitivity and limit access accordingly.

Potential data includes:

- Account identifiers and profile information.
- Session records.
- Friends, parties, and presence.
- Match history and ranking.
- Cosmetic ownership and progression.
- Chat and moderation records.
- Reports and audit logs.

Requirements:

- Collect only information needed to operate the game.
- Restrict database permissions to the minimum necessary.
- Validate ownership before returning private account data.
- Do not expose session records, private moderation notes, or internal security signals to other players.
- Use encrypted transport for production connections.
- Protect database backups and any stored evidence.
- Define retention periods for chat, reports, logs, and inactive accounts.
- Provide a reasonable process for account deletion and applicable data requests.

Do not invent retention periods before considering operational, safety, and legal requirements.

## 11. Secrets and Environment Configuration

Secrets must remain on the server.

Examples include database credentials, authentication secrets, signing keys, and provider API keys.

- Store secrets in environment variables or the selected platform's secret manager.
- Do not commit real secrets to Git.
- Do not expose server secrets through public environment variables or client bundles.
- Keep local development secrets out of tracked files.
- Use separate credentials for development and production.
- Rotate credentials if they are exposed.
- Avoid printing secrets in build output, logs, or error responses.

Public client configuration, such as a public API base URL, is not a secret and should be treated differently.

## 12. Web Application Security

The web client and its HTTP API should apply appropriate protections for their architecture.

Review:

- CORS allowlists.
- Session and cookie configuration.
- Cross-site request forgery protections where applicable.
- Cross-site scripting prevention.
- Input validation and output encoding.
- Safe file upload handling if uploads are introduced.
- Dependency vulnerabilities.
- Security headers where supported.
- Error handling that does not expose stack traces or credentials in production.

CORS is not authentication. A permitted browser origin does not make a request trustworthy, and non-browser clients can often bypass browser-origin restrictions.

Real-time connections require their own origin, authentication, authorization, and resource-limit checks.

## 13. Logging and Incident Response

Security logging should help investigate abuse without creating unnecessary privacy risks.

Record events such as:

- Repeated invalid gameplay commands.
- Authentication failures and suspicious session activity.
- Unauthorized room-access attempts.
- Rate-limit enforcement.
- Moderation decisions.
- Administrative permission changes.
- Unexpected match or server failures.

Do not log passwords, raw session tokens, or unnecessary personal data.

Establish a basic incident process:

1. Identify and assess the issue.
2. Limit ongoing harm.
3. Preserve relevant evidence.
4. Revoke compromised credentials or access where necessary.
5. Apply a fix and verify it.
6. Review the cause and improve safeguards.

For serious incidents, document what happened and what was changed without exposing affected users' private information.

## 14. Testing Requirements

Security tests should cover both ordinary mistakes and deliberate abuse.

Minimum cases include:

- A client attempts to forge damage or score.
- A player sends malformed or oversized payloads.
- A player attempts to join a restricted room.
- A user attempts to perform moderator actions.
- A player sends messages after being restricted.
- A player floods a chat or real-time endpoint.
- A stale or revoked session attempts to continue acting.
- A client attempts to impersonate another account.
- A disconnected player attempts to retain match state.
- User-generated content attempts to execute as HTML or script.

Test authorization and gameplay rules at the server boundary. Client-only tests cannot establish that these protections work.

## 15. Implementation Priorities

### Before the first public multiplayer test

- Establish server-authoritative combat and scoring.
- Validate real-time messages.
- Enforce session identity and room permissions.
- Add basic rate limits and resource limits.
- Keep secrets out of client code and Git.
- Handle disconnections and cleanup.
- Add tests for forged gameplay actions.

### Before broader public access

- Complete reporting and moderation workflows.
- Add protected administrative roles and audit logs.
- Add operational and security monitoring.
- Review dependency vulnerabilities.
- Test reconnect, spam, and abuse scenarios.
- Document account suspension and appeal procedures.

### As the game grows

- Improve suspicious-behavior detection.
- Review data retention and account-deletion workflows.
- Test higher connection volumes.
- Reassess permissions and administrative access.
- Conduct a deeper security review before major competitive or economic features.

## 16. Open Decisions

- [ ] Verify the current authentication implementation.
- [ ] Confirm the multiplayer framework and connection-authentication flow.
- [ ] Define the final role and permission model.
- [ ] Define account suspension, permanent bans, and appeals.
- [ ] Establish chat and moderation retention rules.
- [ ] Decide which match evidence can be retained for reports.
- [ ] Define account deletion and data-request handling.
- [ ] Select production secret management and deployment controls.
- [ ] Determine which suspicious behaviors should trigger automated restrictions.

## 17. Definition of Done

This document is ready to guide implementation when the team can verify that:

- Gameplay outcomes are controlled by the server.
- Untrusted inputs are validated.
- Account identity and room membership are checked server-side.
- Privileged operations enforce authorization.
- Chat and gameplay endpoints have appropriate limits.
- Reports and moderation actions are protected.
- Secrets are not exposed in the browser or committed to the repository.
- Security-relevant behavior has automated tests.
- Operational logs support investigation without unnecessarily exposing private data.

**Security rule:** Every important action must be authorized and validated by the system that owns the data or game state, not merely by the interface requesting it.