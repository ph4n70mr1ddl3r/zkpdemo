---
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - docs/prd.md
  - docs/architecture.md
---

# Headsup - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for Headsup, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

FR1: Users can create an account with username and password
FR2: Users can log in with their credentials to receive a session token
FR3: Users can log out, invalidating their session
FR4: The system maintains session state for authenticated users
FR5: Users have a play money chip balance associated with their account
FR6: Users can view a list of available heads-up poker tables
FR7: Users can see table information (stakes, players, game status)
FR8: Users can join an available table
FR9: Users can leave a table they are seated at
FR10: The system supports multiple concurrent tables running simultaneously
FR11: The system deals a complete heads-up No Limit Hold'em game
FR12: Players receive two hole cards at the start of each hand
FR13: The system deals community cards (flop, turn, river) at appropriate times
FR14: Players can perform betting actions (fold, check, call, raise, all-in)
FR15: The system enforces valid betting actions based on game state
FR16: The system manages betting rounds (pre-flop, flop, turn, river)
FR17: The system calculates pot size including side pots for all-in scenarios
FR18: The system evaluates poker hands at showdown to determine winners
FR19: The system awards chips to winning players
FR20: The system manages player chip stacks throughout the game
FR21: The system detects when a player disconnects during a game
FR22: Players can reconnect to an in-progress game after disconnection
FR23: The system preserves game state during player disconnections
FR24: The system implements timeout mechanisms for inactive players
FR25: The system handles partial connections (one player connected, one disconnected)
FR26: The system validates all player actions server-side before executing them
FR27: The system rejects invalid or malformed game actions
FR28: The system enforces rate limiting on player actions
FR29: The system prevents multiple concurrent connections from the same player account
FR30: The system logs suspicious behavior patterns for review
FR31: The system maintains authoritative game state on the server
FR32: The system broadcasts game state updates to connected players
FR33: The system ensures game state transitions follow poker rules
FR34: The system prevents invalid state transitions
FR35: The system guarantees state integrity under all conditions (disconnections, errors, malicious actions)
FR39: The system returns appropriate error messages for invalid requests

### NonFunctional Requirements

NFR1: Game actions (fold, call, raise) complete with sub-100ms server-side processing time
NFR2: WebSocket message round-trip latency stays below 100ms under normal network conditions
NFR3: Lobby operations (list tables, join table) complete within 1 second
NFR4: Server supports minimum 20 concurrent poker games without performance degradation
NFR5: Server handles game state updates and broadcasts to all active players within 50ms
NFR6: System gracefully degrades under load by rejecting new connections rather than crashing
NFR7: Server maintains performance targets up to configured capacity limits
NFR8: Session tokens use cryptographically secure random generation (minimum 256-bit entropy)
NFR9: Passwords stored using secure hashing algorithms (bcrypt, argon2, or equivalent)
NFR10: Session tokens expire after configurable inactivity period
NFR11: All authentication attempts logged for security monitoring
NFR12: All game-critical logic executes server-side with zero trust in client data
NFR13: Server validates all incoming messages for structure, content, and game legality
NFR14: Session hijacking prevented through secure token management and validation
NFR15: Rate limiting enforces maximum requests per second per connection
NFR16: Malformed or invalid actions rejected without terminating connection (unless repeated violations)
NFR17: Suspicious behavior patterns logged and flagged for review
NFR18: Multiple concurrent connections from same account prevented
NFR19: Server handles player disconnections gracefully without corrupting game state
NFR20: Players can reconnect to in-progress games after network interruption
NFR21: Game state persisted across disconnections to enable recovery
NFR22: All game state transitions validated against poker rules before execution
NFR23: Invalid state transitions prevented through state machine design
NFR24: Zero state corruption tolerance - property-based testing validates state integrity across random scenarios
NFR25: Game state remains consistent under all conditions (disconnections, errors, malicious actions)
NFR26: All errors return structured error messages with error codes and descriptions
NFR27: Server continues operating normally when individual games encounter errors
NFR28: Errors logged with sufficient detail for debugging and monitoring
NFR29: Comprehensive test suite achieves high coverage of game logic and edge cases
NFR30: Property-based tests validate game logic across thousands of random scenarios
NFR31: Fuzz testing discovers edge cases not explicitly anticipated
NFR32: Security validation includes penetration testing and malicious player simulation
NFR33: Code follows consistent style and naming conventions
NFR34: Critical game logic includes comprehensive inline documentation
NFR35: State machine design enables reasoning about valid transitions
NFR36: Server logs critical events (authentication, game actions, errors, security events)
NFR37: Logging provides sufficient detail for debugging production issues
NFR38: System provides visibility into concurrent game count and connection status

### Additional Requirements

- **Starter Template**: Use `cargo init --name headsup` for project initialization (Architecture)
- **Database Migrations**: Use `rusqlite_migration` library for automated database migration management (Architecture)
- **WebSocket Message Protocol**: Use envelope pattern with strongly-typed Rust enums (Architecture)
- **Game State Management**: In-Memory State with Periodic Database Sync (Architecture)
- **Session Storage**: In-Memory HashMap for MVP (Architecture)
- **Rate Limiting**: Simple Per-Connection Counter (Architecture)
- **Logging Framework**: Use `tracing` crate with `tracing-subscriber` (Architecture)
- **Testing Framework**: Use `proptest` for property-based testing (Architecture)
- **Error Handling**: Use `thiserror` for custom error types (Architecture)
- **Project Structure**: Follow defined module organization (game/, auth.rs, db.rs, etc.) (Architecture)
- **Logging Levels**: Enforce specific log levels (ERROR, WARN, INFO, DEBUG, TRACE) (Architecture)

### FR Coverage Map

FR1: Epic 1 - Account Creation
FR2: Epic 1 - Login & Session Token
FR3: Epic 1 - Logout
FR4: Epic 1 - Session State Maintenance
FR5: Epic 1 - Chip Balance (Initial setup)
FR6: Epic 2 - List Tables
FR7: Epic 2 - View Table Info
FR8: Epic 2 - Join Table
FR9: Epic 2 - Leave Table
FR10: Epic 2 - Concurrent Tables Support
FR11: Epic 3 - Deal Complete Hand
FR12: Epic 3 - Deal Hole Cards
FR13: Epic 3 - Deal Community Cards
FR14: Epic 3 - Betting Actions (Fold/Call/Raise/Check/All-in)
FR15: Epic 3 - Validate Betting Actions
FR16: Epic 3 - Manage Betting Rounds
FR17: Epic 3 - Pot Calculation (Side Pots)
FR18: Epic 3 - Hand Evaluation
FR19: Epic 3 - Award Chips
FR20: Epic 3 - Chip Stack Management
FR21: Epic 4 - Detect Disconnect
FR22: Epic 4 - Reconnection
FR23: Epic 4 - Preserve State on Disconnect
FR24: Epic 4 - Player Timeout
FR25: Epic 4 - Handle Partial Connections
FR26: Epic 4 - Server-Side Action Validation (Security)
FR27: Epic 4 - Reject Invalid Actions
FR28: Epic 4 - Rate Limiting
FR29: Epic 1 - Prevent Multi-Connections (Account level)
FR30: Epic 4 - Log Suspicious Behavior
FR31: Epic 3 - Authoritative Server State
FR32: Epic 3 - Broadcast State Updates
FR33: Epic 3 - Enforce Poker Rules
FR34: Epic 3 - Prevent Invalid Transitions
FR35: Epic 4 - Guarantee State Integrity (Resilience)
FR39: All Epics - Error Messaging (distributed)

## Epic List

### Epic 1: Foundation & Identity (User Auth & Session Management)
Users can securely create accounts, log in, and establish a persistent session to interact with the system.
**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR29, FR39

### Epic 2: Lobby & Multiplayer Connectivity
Users can discover active games, join tables, and see other players in a real-time environment.
**FRs covered:** FR6, FR7, FR8, FR9, FR10, FR32, FR39

### Epic 3: Core Poker Game Engine
Users can play a complete hand of Heads-Up NLHE, from dealing cards to determining a winner, with correct rule enforcement.
**FRs covered:** FR11, FR12, FR13, FR14, FR15, FR16, FR17, FR18, FR19, FR20, FR31, FR33, FR34

### Epic 4: Game Integrity & Resilience
Users experience a fair and robust game that handles interruptions, disconnects, and edge cases gracefully without state corruption.
**FRs covered:** FR21, FR22, FR23, FR24, FR25, FR26, FR27, FR28, FR30, FR35

## Epic 1: Foundation & Identity (User Auth & Session Management)

Users can securely create accounts, log in, and establish a persistent session to interact with the system.

### Story 1.1: User Registration (Signup)

As a New User,
I want to create an account with a username and password,
So that I can identify myself and save my progress.

**Acceptance Criteria:**

**Given** The server is initialized (database & basic logging setup)
**When** A client sends a registration request with a unique username and valid password
**Then** A new user record is created in the database with a secure password hash (bcrypt)
**And** The user is initialized with a starting chip balance of 1000 (FR5)
**And** The server returns a success response with the new user ID
**But** If the username already exists, the server returns a "Username taken" error

### Story 1.2: User Login & Session Creation

As a Registered User,
I want to log in with my credentials,
So that I can receive a secure session token to access the game.

**Acceptance Criteria:**

**Given** A registered user exists in the database
**When** A client sends a login request with correct username and password
**Then** The server verifies the password hash
**And** A cryptographically secure session token (min 256-bit) is generated (NFR8)
**And** The session is stored in the in-memory session store mapped to the user ID
**And** The server returns the session token to the client
**And** The client can subsequently authenticate the WebSocket connection using this token (handshake)
**But** If credentials are invalid, the server returns an "Invalid credentials" error

### Story 1.3: Logout & Session Invalidation

As a Logged-in User,
I want to log out of my account,
So that my session is invalidated and no one else can use it.

**Acceptance Criteria:**

**Given** An authenticated user with an active session
**When** The client sends a `Logout` message
**Then** The session token is removed from the in-memory store
**And** The WebSocket connection is closed
**And** Subsequent attempts to use that token result in authentication errors

## Epic 2: Lobby & Multiplayer Connectivity

Users can discover active games, join tables, and see other players in a real-time environment.

### Story 2.1: List Available Tables

As a Player,
I want to view a list of all available tables,
So that I can choose a game to join.

**Acceptance Criteria:**

**Given** An authenticated user connected via WebSocket
**And** The Lobby System is initialized with active tables (supporting 20+ concurrent tables)
**When** The user sends a `ListTables` message
**Then** The server responds with a list of all active tables
**And** The response includes Table ID, stakes, current player count (0/2, 1/2), and status for each table
**And** The operation completes in under 1 second (NFR3)

### Story 2.2: Join Table

As a Player,
I want to join a specific table,
So that I can sit down and prepare to play.

**Acceptance Criteria:**

**Given** An authenticated user not currently seated at a table
**And** A table exists with an open seat (player count < 2)
**When** The user sends a `JoinTable` message with the Table ID
**Then** The user is assigned to an empty seat at that table
**And** The table's player count is updated
**And** The server broadcasts a `PlayerJoined` event to all players at that table
**But** If the table is full, the server returns a "Table Full" error
**But** If the user is already seated elsewhere, the server returns an error

### Story 2.3: Leave Table

As a Seated Player,
I want to leave my current table,
So that I can stop playing or join a different table.

**Acceptance Criteria:**

**Given** A user is currently seated at a table
**When** The user sends a `LeaveTable` message
**Then** The user is removed from the seat
**And** The seat becomes available for others
**And** The server broadcasts a `PlayerLeft` event to any remaining player at the table
**And** The user is now free to join other tables

## Epic 3: Core Poker Game Engine

Users can play a complete hand of Heads-Up NLHE, from dealing cards to determining a winner, with correct rule enforcement.

### Story 3.1: Game Hand Progression

As a Player,
I want the game to progress automatically through standard poker phases (deal, bet, showdown),
So that I can play a complete hand of poker.

**Acceptance Criteria:**

**Given** A table with 2 seated players having sufficient chips
**And** A correctly implemented deck shuffling mechanism (randomized & unique cards)
**When** The game starts
**Then** The state transitions to `PreFlop` and blinds are posted
**And** Hole cards are dealt to both players
**And** The game correctly handles transitions to Flop, Turn, and River after betting rounds
**And** The `evaluate_hand` logic correctly identifies the winner at Showdown
**And** Game state updates are broadcast to both players (FR32)

### Story 3.2: Betting Logic (Pre-Flop)

As a Player,
I want to perform betting actions (Fold, Call, Check, Raise, All-in),
So that I can compete for the pot.

**Acceptance Criteria:**

**Given** It is my turn in the `PreFlop` phase
**When** I send a valid betting action (e.g., `Call`)
**Then** My chip stack decreases by the correct amount
**And** The pot size increases
**And** The turn passes to the opponent
**And** Valid actions are enforced (e.g., cannot `Check` if facing a bet)
**But** If I try to act out of turn, the action is rejected

### Story 3.3: Post-Flop Betting Rounds

As a Player,
I want the game to proceed to the Flop, Turn, and River,
So that I can see community cards and bet on them.

**Acceptance Criteria:**

**Given** The pre-flop betting round is complete (both players match bets)
**When** The game transitions to the next street (e.g., Flop)
**Then** The correct number of community cards are dealt and revealed
**And** The current round bet is reset to 0
**And** The turn order is reset (Dealer acts last post-flop)
**And** A new betting round begins

### Story 3.4: Pot Management & Side Pots

As a Player,
I want the pot to be calculated correctly even when I go All-in,
So that I only win or lose the amount I wagered.

**Acceptance Criteria:**

**Given** I have fewer chips than my opponent (e.g., 50 vs 100)
**When** I go `All-in` (50) and my opponent `Calls` (matches 50)
**Then** A main pot is formed with 100 chips
**And** My opponent retains their excess chips
**And** The betting round ends (no further betting possible)
**And** The system correctly tracks this all-in state

### Story 3.5: Showdown & Winner Determination

As a Player,
I want the game to determine the winner and award chips,
So that the hand concludes fairly.

**Acceptance Criteria:**

**Given** The final betting round (River) is complete
**When** The game reaches `Showdown`
**Then** Both players' hole cards are revealed
**And** The best 5-card hand is determined for each player
**And** The player with the higher ranking hand is awarded the entire pot
**And** In case of a tie, the pot is split equally
**And** Player chip stacks are updated with winnings
**And** The game waits briefly before starting the next hand

## Epic 4: Game Integrity & Resilience

Users experience a fair and robust game that handles interruptions, disconnects, and edge cases gracefully without state corruption.

### Story 4.1: Disconnection Detection & Handling

As a Player,
I want the game to handle it gracefully if my opponent disconnects,
So that the game doesn't crash or hang indefinitely.

**Acceptance Criteria:**

**Given** A game is in progress
**When** My opponent's WebSocket connection drops
**Then** The server detects the disconnection immediately
**And** The opponent's status is updated to `Disconnected`
**And** I receive a notification that my opponent has disconnected
**And** The game pauses or starts a reconnection timer (depending on game phase)

### Story 4.2: Game Reconnection & State Recovery

As a Player,
I want to be able to rejoin my game after a brief internet outage,
So that I don't lose my chips or my seat.

**Acceptance Criteria:**

**Given** I was disconnected from an active game
**When** I reconnect to the server with my valid session token
**Then** The server recognizes I am part of an active game
**And** I am automatically re-subscribed to the game updates
**And** I receive the full current state of the game (cards, pot, board, turn)
**And** I can immediately resume playing my hand

### Story 4.3: Fair Play & Security

As a Player,
I want the game to reject invalid actions and prevent spam,
So that cheaters cannot ruin the game experience.

**Acceptance Criteria:**

**Given** A user attempts to send invalid data (e.g. negative bets, huge payloads) or flood the server
**When** The input validation layer receives the message
**Then** It rejects the malformed/invalid message before it reaches game logic
**And** Rate limiting prevents the user from spamming the server (>10 msg/sec)
**And** The server logs the suspicious attempt
**And** The game state remains completely unchanged

### Story 4.4: Action Timer & Player Timeout

As a Player,
I want the game to proceed if my opponent stops acting,
So that I'm not stuck waiting forever.

**Acceptance Criteria:**

**Given** It is my opponent's turn
**When** They fail to act within the defined time limit (e.g., 30 seconds)
**Then** The server automatically performs a default action for them
**And** The default action is `Check` if there is no bet to call, or `Fold` if there is a bet
**And** The game proceeds to the next turn or hand
