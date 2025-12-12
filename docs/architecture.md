---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments: 
  - "\\\\wsl.localhost\\riddler\\home\\riddler\\headsup\\docs\\prd.md"
workflowType: 'architecture'
lastStep: 5
project_name: 'Headsup'
user_name: 'Riddler'
date: '2025-12-13T06:28:35+08:00'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**

Headsup requires **35 functional capabilities** organized around six core system areas:

1. **User Account Management (FR1-FR5):** Username/password authentication with session-based auth, play money chip balance management
2. **Lobby & Table Discovery (FR6-FR10):** Browse available tables, view table metadata (stakes, players, status), join/leave operations, support for multiple concurrent tables
3. **Poker Game Mechanics (FR11-FR20):** Complete heads-up NLHE implementation including hole card dealing, community cards, betting actions (fold/check/call/raise/all-in), betting round management, pot calculations with side pots, hand evaluation, chip awarding, and stack management
4. **Network Resilience (FR21-FR25):** Disconnection detection, reconnection support, game state preservation during disconnects, timeout mechanisms, partial connection handling
5. **Security & Validation (FR26-FR30):** Server-side action validation, rejection of invalid/malformed actions, rate limiting, prevention of multiple concurrent connections, suspicious behavior logging
6. **Game State Management (FR31-FR35):** Authoritative server-side state, state broadcast to players, rule-enforced state transitions, invalid state prevention, integrity guarantees under all conditions

**Architectural Implications:**
- State machine design is critical - poker has complex state transitions (betting rounds, all-in scenarios, showdown logic)
- WebSocket connection lifecycle must be tightly integrated with game state
- Server must be the single source of truth - no client-side game logic
- Disconnection handling requires state persistence and recovery mechanisms

**Non-Functional Requirements:**

**Performance Requirements (NFR1-NFR7):**
- Sub-100ms server-side processing for game actions
- Sub-100ms WebSocket round-trip latency
- Support minimum 20 concurrent games without degradation
- 50ms broadcast latency for state updates
- Graceful degradation (reject connections vs crash)

**Security Requirements (NFR8-NFR18):**
- Cryptographically secure session tokens (256-bit entropy minimum)
- Secure password hashing (bcrypt/argon2)
- Configurable session expiration
- Zero trust in client data - all logic server-side
- Message validation (structure, content, game legality)
- Rate limiting enforcement
- Malicious action rejection without connection termination
- Suspicious behavior pattern detection
- Single connection per account enforcement

**Reliability Requirements (NFR19-NFR28):**
- Graceful disconnection handling without state corruption
- Reconnection support for in-progress games
- State persistence across disconnections
- Poker rule validation on all state transitions
- Zero state corruption tolerance
- Structured error responses with codes
- Game isolation (individual game errors don't affect server)

**Testability Requirements (NFR29-NFR32):**
- Comprehensive test coverage
- Property-based testing across thousands of random scenarios
- Fuzz testing for edge case discovery
- Security penetration testing and malicious player simulation

**Architectural Implications:**
- Real-time performance demands efficient WebSocket and state management implementations
- Security requirements necessitate defense-in-depth architecture
- Testability focus suggests backend-only MVP was wise choice
- Zero corruption tolerance requires robust state machine with invariant checking

### Scale & Complexity

- **Primary domain:** Real-time Backend Gaming Server
- **Complexity level:** Medium-High
  - **Medium:** Heads-up only (simpler than full-table), play money (no financial compliance)
  - **High:** Real-time multiplayer coordination, complex poker state machine, network resilience requirements, security hardening
- **Estimated architectural components:** 8-10 major components
  - WebSocket connection management
  - Authentication & session management
  - Lobby system
  - Game state machine (poker logic core)
  - Hand evaluator
  - Player action validation
  - State persistence & recovery
  - Rate limiting & security
  - Logging & monitoring
  - Message protocol handling

### Technical Constraints & Dependencies

**Technology Constraints:**
- **Communication Protocol:** WebSocket with JSON messaging (specified in PRD)
- **Authentication Model:** Session-based with token authentication (specified in PRD)
- **No API Versioning:** Single protocol version for MVP with synchronous client/server upgrades
- **No SDK Provided:** Protocol documented via JSON schemas

**Deployment Constraints:**
- Must support concurrent connection handling (async/event-driven architecture implied)
- Session storage backend needed (in-memory for MVP, database-backed for production)
- Sub-100ms latency requirements suggest single-region deployment for MVP

**Testing Constraints:**
- Backend-only MVP enables comprehensive automated testing
- Property-based and fuzz testing required for game logic validation
- Security testing must include malicious player simulation

### Cross-Cutting Concerns Identified

**1. WebSocket Connection Management**
- Affects: Authentication, Lobby, Game Play, State Synchronization
- Requirements: Persistent connections, reconnection handling, graceful degradation
- Architecture impact: Need robust connection lifecycle management integrated throughout

**2. Server-Side Validation & Security**
- Affects: All player actions, all state transitions
- Requirements: Zero trust model, rate limiting, malicious actor prevention
- Architecture impact: Validation layer needed across all components; security cannot be afterthought

**3. State Persistence & Recovery**
- Affects: Game state, Player state, Session state
- Requirements: Survive disconnections, enable reconnection, maintain integrity
- Architecture impact: Persistence layer design critical; state serialization/deserialization needed

**4. Real-Time State Synchronization**
- Affects: All game actions, lobby updates
- Requirements: Sub-50ms broadcast latency, consistency across clients
- Architecture impact: Efficient state broadcasting mechanism; careful consideration of what to push vs pull

**5. Logging & Observability**
- Affects: All components
- Requirements: Security event logging, game action audit trail, debugging capability, operational visibility
- Architecture impact: Structured logging framework; clear logging standards across components

**6. Testing Infrastructure**
- Affects: All components
- Requirements: Unit, integration, property-based, fuzz, and security testing
- Architecture impact: Design for testability; clear component boundaries; game logic isolated for property-based testing

**7. Error Handling & Resilience**
- Affects: All components
- Requirements: Structured errors with codes, game isolation, graceful degradation
- Architecture impact: Standard error handling patterns; bulkhead pattern for game isolation

## Starter Template Evaluation

### Primary Technology Domain

**Real-time Backend WebSocket Server** built with Rust, targeting beginner-friendly setup with production-ready patterns.

### Starter Options Considered

**Option 1: From Scratch with `cargo init`**
- **Pros:** Complete control, learn Rust project structure from ground up, no unnecessary dependencies
- **Cons:** Must manually configure all dependencies, no WebSocket boilerplate, more initial setup work

**Option 2: Actix-web WebSocket Template** (GitHub examples)
- **Pros:** Proven high-performance framework, extensive documentation, active community
- **Cons:** No official CLI starter, must copy example code manually

**Option 3: Custom Foundation** (Selected for beginners)
- **Pros:** Starts simple with `cargo init`, adds dependencies incrementally, beginner-friendly learning curve
- **Cons:** Requires more guidance through architecture document

### Selected Starter: Manual Initialization with Guided Architecture

**Rationale for Selection:**
- Rust doesn't have a dominant "create-react-app" equivalent for WebSocket servers
- `cargo init` provides clean, idiomatic Rust project structure
- Manual dependency selection teaches Rust ecosystem and avoids bloat
- Perfect for learning while building production-ready foundations
- Aligns with "foundation-first" philosophy from PRD

**Initialization Command:**

```bash
cargo init --name headsup
```

### Architectural Decisions Provided by Starter:

**Language & Runtime:**
- **Rust Edition:** 2024 (current default)
- **Async Runtime:** `tokio` with full features (industry standard for async Rust)
- **WebSocket Library:** `tokio-tungstenite` (maintained, efficient, integrates with tokio)
- **Web Framework:** `actix-web` + `actix-ws` (high performance, beginner-friendly API)

**Database & Persistence:**
- **SQLite Library:** `rusqlite` with bundled feature (recommended for simplicity and synchronous operations)
- **Migrations:** `rusqlite_migration` or manual SQL scripts in `migrations/` directory

**Core Dependencies:**
```toml
[dependencies]
actix-web = "4"
actix-ws = "0.3"
tokio = { version = "1", features = ["full"] }
tokio-tungstenite = "0.21"
rusqlite = { version = "0.31", features = ["bundled"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
bcrypt = "0.15"    # Password hashing
rand = "0.8"       # Session token generation
thiserror = "1"    # Error handling
```

**Build Tooling:**
- **Build Command:** `cargo build` (development), `cargo build --release` (production)
- **Run Command:** `cargo run`
- **Test Command:** `cargo test`
- **Development Tools:** `cargo-watch` for file change monitoring
- **Linting:** `rustfmt` (formatting), `clippy` (linting)

**Testing Framework:**
- **Unit Tests:** Built-in Rust `#[test]` with `#[cfg(test)]` modules
- **Property-Based Testing:** `proptest` for game logic validation (thousands of random scenarios)
- **Async Testing:** `tokio::test` macro for async test cases
- **Integration Tests:** `tests/` directory for end-to-end testing

**Code Organization:**

```
headsup/
├── Cargo.toml
├── Cargo.lock
├── src/
│   ├── main.rs           # Entry point, server initialization
│   ├── websocket.rs      # WebSocket connection handling
│   ├── game/
│   │   ├── mod.rs        # Game state machine
│   │   ├── poker.rs      # Poker logic (hand evaluation, etc.)
│   │   └── deck.rs       # Card dealing
│   ├── lobby.rs          # Lobby management
│   ├── auth.rs           # Authentication & session management
│   ├── db.rs             # Database operations (rusqlite)
│   ├── models.rs         # Data structures (Player, Table, GameState, etc.)
│   ├── validation.rs     # Server-side validation & rate limiting
│   └── errors.rs         # Custom error types
├── tests/
│   └── integration_tests.rs
└── migrations/
    └── 001_initial_schema.sql
```

**Project Structure Rationale:**
- **Modules:** Clear separation of concerns (auth, game, lobby, db) for maintainability
- **Error Handling:** Custom error types with `thiserror` for type-safe error propagation
- **State Management:** Actix-web's shared state with `Arc<Mutex<T>>` for game state synchronization
- **Message Protocol:** Strongly-typed enums with `serde` for JSON serialization/deserialization

**Development Experience:**
- **Hot Reloading:** `cargo install cargo-watch`, then `cargo watch -x run`
- **Debugging:** `rust-gdb` or IDE integration (VS Code with rust-analyzer extension)
- **Documentation:** `cargo doc --open` generates and opens documentation
- **Benchmarking:** `cargo bench` with `criterion` crate for performance testing

**Deployment Configuration:**
- **Target Platform:** Linux VPS (production) / WSL (testing)
- **Build Optimization:** `cargo build --release` produces optimized binary (10-100x faster than debug)
- **SQLite Database:** File-based (`headsup.db`), bundled with rusqlite for portability
- **Environment Variables:** `dotenv` crate for `.env` file support (database path, session secrets, etc.)
- **Logging:** `env_logger` or `tracing` for structured logging

**Note:** Project initialization using `cargo init --name headsup` should be the first implementation story.

- Message validation (structure, content, game legality)
- Rate limiting enforcement
- Malicious action rejection without connection termination
- Suspicious behavior pattern detection
- Single connection per account enforcement

**Reliability Requirements (NFR19-NFR28):**
- Graceful disconnection handling without state corruption
- Reconnection support for in-progress games
- State persistence across disconnections
- Poker rule validation on all state transitions
- Zero state corruption tolerance
- Structured error responses with codes
- Game isolation (individual game errors don't affect server)

**Testability Requirements (NFR29-NFR32):**
- Comprehensive test coverage
- Property-based testing across thousands of random scenarios
- Fuzz testing for edge case discovery
- Security penetration testing and malicious player simulation

**Architectural Implications:**
- Real-time performance demands efficient WebSocket and state management implementations
- Security requirements necessitate defense-in-depth architecture
- Testability focus suggests backend-only MVP was wise choice
- Zero corruption tolerance requires robust state machine with invariant checking

### Scale & Complexity

- **Primary domain:** Real-time Backend Gaming Server
- **Complexity level:** Medium-High
  - **Medium:** Heads-up only (simpler than full-table), play money (no financial compliance)
  - **High:** Real-time multiplayer coordination, complex poker state machine, network resilience requirements, security hardening
- **Estimated architectural components:** 8-10 major components
  - WebSocket connection management
  - Authentication & session management
  - Lobby system
  - Game state machine (poker logic core)
  - Hand evaluator
  - Player action validation
  - State persistence & recovery
  - Rate limiting & security
  - Logging & monitoring
  - Message protocol handling

### Technical Constraints & Dependencies

**Technology Constraints:**
- **Communication Protocol:** WebSocket with JSON messaging (specified in PRD)
- **Authentication Model:** Session-based with token authentication (specified in PRD)
- **No API Versioning:** Single protocol version for MVP with synchronous client/server upgrades
- **No SDK Provided:** Protocol documented via JSON schemas

**Deployment Constraints:**
- Must support concurrent connection handling (async/event-driven architecture implied)
- Session storage backend needed (in-memory for MVP, database-backed for production)
- Sub-100ms latency requirements suggest single-region deployment for MVP

**Testing Constraints:**
- Backend-only MVP enables comprehensive automated testing
- Property-based and fuzz testing required for game logic validation
- Security testing must include malicious player simulation

### Cross-Cutting Concerns Identified

**1. WebSocket Connection Management**
- Affects: Authentication, Lobby, Game Play, State Synchronization
- Requirements: Persistent connections, reconnection handling, graceful degradation
- Architecture impact: Need robust connection lifecycle management integrated throughout

**2. Server-Side Validation & Security**
- Affects: All player actions, all state transitions
- Requirements: Zero trust model, rate limiting, malicious actor prevention
- Architecture impact: Validation layer needed across all components; security cannot be afterthought

**3. State Persistence & Recovery**
- Affects: Game state, Player state, Session state
- Requirements: Survive disconnections, enable reconnection, maintain integrity
- Architecture impact: Persistence layer design critical; state serialization/deserialization needed

**4. Real-Time State Synchronization**
- Affects: All game actions, lobby updates
- Requirements: Sub-50ms broadcast latency, consistency across clients
- Architecture impact: Efficient state broadcasting mechanism; careful consideration of what to push vs pull

**5. Logging & Observability**
- Affects: All components
- Requirements: Security event logging, game action audit trail, debugging capability, operational visibility
- Architecture impact: Structured logging framework; clear logging standards across components

**6. Testing Infrastructure**
- Affects: All components
- Requirements: Unit, integration, property-based, fuzz, and security testing
- Architecture impact: Design for testability; clear component boundaries; game logic isolated for property-based testing

**7. Error Handling & Resilience**
- Affects: All components
- Requirements: Structured errors with codes, game isolation, graceful degradation
- Architecture impact: Standard error handling patterns; bulkhead pattern for game isolation

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Database schema and migrations strategy
- WebSocket message protocol structure
- Game state management approach
- Session storage mechanism
- Rate limiting implementation
- Logging framework selection

All critical architectural decisions have been made collaboratively and documented below.

### Data Architecture

**Database Migrations: `rusqlite_migration` crate**

**Decision:** Use `rusqlite_migration` library for automated database migration management

**Rationale:**
- Automated migration tracking eliminates manual error-prone tracking
- Embedded migrations in binary simplify deployment
- Rollback support provides safety during schema evolution
- Sequential numbered migrations (001_initial_schema.sql, 002_add_sessions.sql, etc.) provide clear versioning
- Well-suited for evolving project (hand history, player stats, tournament support later)

**Implementation:**
```toml
rusqlite_migration = "1.0"
```

**Migration Strategy:**
- Store migrations in `/migrations` directory
- Sequential numbering: `001_description.sql`, `002_description.sql`, etc.
- Apply migrations on server startup
- Track applied migrations in `__schema_migrations` table

**Affects:** Database initialization, deployment procedures, version upgrades

---

### API & Communication Patterns

**WebSocket Message Protocol: Envelope Pattern**

**Decision:** Use envelope pattern for all WebSocket messages with strongly-typed Rust enums

**Message Structure:**
```json
{
  "type": "MessageType",
  "payload": { ... },
  "timestamp": "ISO 8601 timestamp"
}
```

**Rationale:**
- Clear message routing based on `type` field
- Easy to extend with metadata (request IDs, correlation IDs for debugging)
- Industry standard pattern for WebSocket communication
- Clean separation between message envelope and payload
- Supports future features (message versioning, tracing, auditing)

**Rust Implementation:**
```rust
#[derive(Serialize, Deserialize)]
struct MessageEnvelope<T> {
    #[serde(rename = "type")]
    message_type: String,
    payload: T,
    timestamp: String,
}

#[derive(Serialize, Deserialize)]
enum ClientMessage {
    Authenticate { session_token: String },
    JoinTable { table_id: u64 },
    LeaveTable,
    Fold,
    Check,
    Call,
    Raise { amount: u64 },
    AllIn,
}

#[derive(Serialize, Deserialize)]
enum ServerMessage {
    AuthSuccess { player_id: u64 },
    AuthError { reason: String },
    GameState { state: GameStateData },
    PlayerAction { player_id: u64, action: String },
    HandComplete { winner: u64, pot: u64 },
    Error { code: String, message: String },
}
```

**Error Message Format:**
```json
{
  "type": "Error",
  "payload": {
    "code": "INVALID_ACTION",
    "message": "Cannot raise when not your turn",
    "details": { "current_turn": "player_123" }
  },
  "timestamp": "2025-12-13T06:45:00Z"
}
```

**Affects:** All client-server communication, WebSocket handlers, message validation

---

### Game State Management

**Approach: In-Memory State with Periodic Database Sync**

**Decision:** Maintain active game state in memory with  async synchronization to SQLite for persistence

**Architecture:**
```rust
// Shared state in Actix-web
struct AppState {
    games: Arc<Mutex<HashMap<GameId, GameState>>>,
    sessions: Arc<Mutex<HashMap<SessionToken, PlayerId>>>,
    db_pool: Arc<Mutex<Connection>>,  // rusqlite connection
}
```

**State Management Strategy:**
- **Active Games:** Stored in `Arc<Mutex<HashMap<GameId, GameState>>>` for fast access
- **Critical Events:** Immediate DB sync (hand completion, chip transfers, player joins/leaves)
- **Non-Critical Events:** Periodic sync every 30 seconds (game state snapshots)
- **Crash Recovery:** Reconstruct active games from last DB snapshot on server restart

**Rationale:**
- **Performance:** In-memory operations achieve sub-1ms latency (far below 100ms NFR requirement)
- **Consistency:** Critical events synced immediately prevent chip/state loss
- **Simplicity:** Simpler than distributed state management for MVP scale (20 concurrent games)
- **Reliability:** Periodic snapshots provide crash recovery without complex WAL implementation

**Trade-offs Accepted:**
- Potential loss of recent non-critical state on crash (acceptable for MVP)
- Memory usage grows with concurrent games (manageable at target scale)
- Future optimization: Can migrate to more sophisticated persistence if needed

**Affects:** Game logic, WebSocket handlers, database layer, server initialization/shutdown

---

### Authentication & Security

**Session Storage: In-Memory HashMap (MVP)**

**Decision:** Store session tokens in-memory for MVP with migration path to database-backed sessions

**Implementation:**
```rust
struct SessionStore {
    sessions: Arc<Mutex<HashMap<SessionToken, SessionData>>>,
}

struct SessionData {
    player_id: u64,
    created_at: SystemTime,
    last_activity: SystemTime,
}
```

**Session Management:**
- **Token Generation:** Cryptographically secure random 256-bit tokens using `rand::thread_rng()`
- **Storage:** In-memory `HashMap<SessionToken, SessionData>`
- **Expiration:** 7 days of inactivity (configurable via environment variable)
- **Cleanup:** Background task expires inactive sessions every hour

**Rationale:**
- **MVP Simplicity:** Fast lookups, simple implementation for initial development
- **Performance:** Zero database overhead on every request
- **Acceptable Trade-off:** Sessions lost on restart (users re-authenticate)

**Future Migration Path:**
- Add `sessions` table to SQLite schema
- Persist sessions to DB on creation/activity
- Load active sessions on server startup
- Maintain same API for session validation

**Security Considerations:**
- Session tokens use `rand::thread_rng()` for cryptographic randomness
- Tokens transmitted only over WebSocket (consider TLS for production)
- Tokens stored as strings (consider hashing for production database storage)

**Affects:** Authentication flow, WebSocket connection handling, player state management

---

**Rate Limiting: Simple Per-Connection Counter**

**Decision:** Implement simple message counter per WebSocket connection with configurable threshold

**Implementation:**
```rust
struct ConnectionState {
    player_id: Option<u64>,
    message_count: AtomicU32,
    last_reset: Mutex<Instant>,
}

// In WebSocket message handler
fn check_rate_limit(conn: &ConnectionState) -> Result<(), RateLimitError> {
    let now = Instant::now();
    let mut last_reset = conn.last_reset.lock().unwrap();
    
    if now.duration_since(*last_reset) >= Duration::from_secs(1) {
        conn.message_count.store(0, Ordering::Relaxed);
        *last_reset = now;
    }
    
    let count = conn.message_count.fetch_add(1, Ordering::Relaxed);
    if count > MAX_MESSAGES_PER_SECOND {
        Err(RateLimitError::ExceededLimit)
    } else {
        Ok(())
    }
}
```

**Configuration:**
- **Threshold:** 10 messages per second per connection (configurable)
- **Window:** 1-second sliding window
- **Response:** Reject excess messages with structured error response
- **No Connection Termination:** Log violations but maintain connection

**Rationale:**
- **MVP Simplicity:** Easy to implement and reason about
- **Effective:** Prevents basic flood attacks and accidental client loops
- **Non-Disruptive:** Doesn't punish users for temporary bursts

**Future Enhancements:**
- Token bucket algorithm for burst allowance
- Per-player global rate limits across all connections
- Automatic temporary bans for repeated violations

**Affects:** WebSocket message handlers, connection state management, error responses

---

### Infrastructure & Deployment

**Logging Framework: `tracing` crate**

**Decision:** Use `tracing` with `tracing-subscriber` for structured, async-aware logging

**Implementation:**
```toml
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
```

**Configuration:**
```rust
use tracing_subscriber::{layer::SubscriberExt, util::SubscriberInitExt};

fn init_logging() {
    tracing_subscriber::registry()
        .with(tracing_subscriber::EnvFilter::new(
            std::env::var("RUST_LOG").unwrap_or_else(|_| "info".into()),
        ))
        .with(tracing_subscriber::fmt::layer())
        .init();
}
```

**Usage Patterns:**
```rust
use tracing::{info, warn, error, debug, instrument};

#[instrument(skip(state))]
async fn handle_player_action(state: &AppState, action: PlayerAction) {
    info!(player_id = action.player_id, action_type = ?action.action_type, "Processing player action");
    // ... logic ...
}
```

**Rationale:**
- **Async-Aware:** Designed for tokio and async Rust (critical for WebSocket server)
- **Structured Logging:** Key-value pairs enable better filtering and analysis
- **Performance:** Minimal overhead in production builds
- **Debugging:** Spans and events help trace async execution across tasks
- **Production Ready:** Industry standard for Rust async applications

**Log Levels:**
- **ERROR:** Unrecoverable errors, state corruption
- **WARN:** Recoverable errors, invalid player actions, rate limit violations
- **INFO:** Connection events, game state transitions, hand completions
- **DEBUG:** Detailed game logic, message processing (development only)
- **TRACE:** Verbose debugging (development only)

**Environment Configuration:**
```bash
# Development
RUST_LOG=debug cargo run

# Production
RUST_LOG=info ./headsup
```

**Affects:** All components (logging is cross-cutting concern), debugging, production monitoring

---

### Decision Impact Analysis

**Implementation Sequence:**
1. **Project initialization:** `cargo init --name headsup`
2. **Database setup:** rusqlite + rusqlite_migration, initial schema
3. **Message protocol:** Define Rust enums for ClientMessage/ServerMessage
4. **Logging setup:** Initialize tracing subscriber in main.rs
5. **Session management:** Implement in-memory session store
6. **Rate limiting:** Add per-connection message counters
7. **Game state:** Implement Arc<Mutex<HashMap>> state management
8. **WebSocket handlers:** Integrate all above components

**Cross-Component Dependencies:**
- **Message Protocol** → affects WebSocket handlers, game logic, client SDK docs
- **Game State Management** → affects database sync, WebSocket broadcasts, crash recovery
- **Session Storage** → affects authentication, WebSocket connection lifecycle
- **Rate Limiting** → affects WebSocket message handlers, error responses
- **Logging** → affects all components (cross-cutting concern)
- **Database Migrations** → affects deployment, version upgrades

## Implementation Patterns & Consistency Rules

### Overview

This section definesimplementation patterns and consistency rules to ensure AI agents write compatible, consistent code. Rust's strong conventions (enforced by `rustfmt` and `clippy`) handle most naming and formatting concerns. This section focuses on **poker-server-specific patterns** where agents could make different choices.

### Rust Language Conventions (Enforced by Tools)

**Naming Conventions (enforced by `rustfmt` and `clippy`):**
- **snake_case:** Functions, variables, modules (e.g., `player_action`, `handle_websocket_message`)
- **PascalCase:** Types, structs, enums, traits (e.g., `GameState`, `ClientMessage`, `PokerCard`)
- **SCREAMING_SNAKE_CASE:** Constants, static variables (e.g., `MAX_PLAYERS`, `DEFAULT_STACK_SIZE`)

**Code Formatting (enforced by `rustfmt`):**
- 4-space indentation
- 100-character line length
- Consistent import grouping

**Linting (enforced by `clippy`):**
- Enable clippy warnings: `#![warn(clippy::all)]`
- Pedantic mode for stricter checks: `#![warn(clippy::pedantic)]`

### Error Handling Patterns

**Custom Error Types with `thiserror`:**

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum GameError {
    #[error("Invalid player action: {0}")]
    InvalidAction(String),
    
    #[error("Game state error: {0}")]
    StateError(String),
    
    #[error("Database error: {0}")]
    DatabaseError(#[from] rusqlite::Error),
    
    #[error("Serialization error: {0}")]
    SerializationError(#[from] serde_json::Error),
}

pub type GameResult<T> = Result<T, GameError>;
```

**Error Handling Rules:**
- **ALL** fallible functions return `Result<T, ErrorType>`
- Use `thiserror` for custom error types with `#[derive(Error)]`
- Use `#[from]` attribute for error conversion from dependencies
- NO `unwrap()` or `expect()` in production code (only in tests or with clear justification)
- Use `?` operator for error propagation
- Log errors at appropriate level before returning them

**Error Response Format (WebSocket):**
```rust
#[derive(Serialize)]
struct ErrorResponse {
    code: String,        // "INVALID_ACTION", "RATE_LIMIT_EXCEEDED", etc.
    message: String,     // Human-readable message
    details: Option<serde_json::Value>,  // Optional context
}
```

### Module Organization Patterns

**Project Structure:**
```
src/
├── main.rs              // Server initialization, tracing setup
├── lib.rs               // Re-exports public API (if creating library crate)
├── models.rs            // Data structures (Player, Table, GameState, Card, etc.)
├── errors.rs            // Custom error types
├── auth.rs              // Authentication & session management
├── db.rs                // Database operations (migrations, CRUD)
├── websocket.rs         // WebSocket connection handling
├── validation.rs        // Input validation & rate limiting
├── game/
│   ├── mod.rs           // Game module re-exports
│   ├── state_machine.rs // Game state machine logic
│   ├── poker.rs         // Poker-specific logic (hand evaluation)
│   └── deck.rs          // Card dealing and deck management
└── constants.rs         // Game constants (blinds, timeouts, etc.)
```

**Module Organization Rules:**
- **Flat modules** for simple components (auth.rs, db.rs)
- **Directory modules** for complex subsystems (game/)
- **Re-export** public items from `mod.rs` for clean API
- **One concept per file:** Don't mix authentication and game logic

### Testing Patterns

**Unit Tests (co-located with code):**

```rust
// In src/game/poker.rs
#[cfg(test)]
mod tests {
    use super::*;
    use proptest::prelude::*;
    
    #[test]
    fn test_evaluate_full_house() {
        let hand = vec![
            Card::new(Rank::King, Suit::Hearts),
            Card::new(Rank::King, Suit::Spades),
            Card::new(Rank::Ace, Suit::Hearts),
            Card::new(Rank::Ace, Suit::Diamonds),
            Card::new(Rank::Ace, Suit::Clubs),
        ];
        assert_eq!(evaluate_hand(&hand), HandRank::FullHouse);
    }
    
    // Property-based test
    proptest! {
        #[test]
        fn test_hand_evaluation_deterministic(
            cards in prop::collection::vec(arbitrary_card(), 5..=7)
        ) {
            let rank1 = evaluate_hand(&cards);
            let rank2 = evaluate_hand(&cards);
            prop_assert_eq!(rank1, rank2);
        }
    }
}
```

**Integration Tests:**
```
tests/
├── integration_tests.rs     // WebSocket connection lifecycle tests
├── game_flow_tests.rs       // End-to-end game scenarios
└── db_tests.rs              // Database integration tests
```

**Testing Rules:**
- Unit tests co-located with code using `#[cfg(test)]` modules
- Integration tests in `tests/` directory
- Use `proptest` for property-based testing of game logic
- Test critical paths: hand evaluation, pot calculation, state transitions
- Mock external dependencies (database, time) in tests

### Database Patterns

**Migration Naming:**
```
migrations/
├── 001_initial_schema.sql
├── 002_add_sessions.sql
├── 003_add_hand_history.sql
└── 004_add_player_stats.sql
```

**Migration Rules:**
- Sequential numbering: `{001-999}_description.sql`
- Descriptive names: `add_sessions`, not `update_schema`
- One logical change per migration
- Include both UP and DOWN migrations (if using reversible migrations)

**SQL Query Organization:**
```rust
// In db.rs
pub struct Database {
    conn: Arc<Mutex<Connection>>,
}

impl Database {
    // Queries as const strings for reusability
    const CREATE_PLAYER: &'static str = r#"
        INSERT INTO players (username, password_hash, chips)
        VALUES (?1, ?2, ?3)
        RETURNING id
    "#;
    
    pub fn create_player(&self, username: &str, password_hash: &str) -> GameResult<u64> {
        let conn = self.conn.lock().unwrap();
        let mut stmt = conn.prepare(Self::CREATE_PLAYER)?;
        let id = stmt.query_row(params![username, password_hash, 1000], |row| row.get(0))?;
        Ok(id)
    }
}
```

**Database Rules:**
- SQL queries as `const` strings for compile-time checking
- Use prepared statements for all queries (SQL injection prevention)
- Wrap `rusqlite::Connection` in `Arc<Mutex<>>` for thread-safe access
- Handle connection errors with `GameError::DatabaseError`

### WebSocket Message Handling Patterns

**Message Deserialization:**
```rust
#[derive(Deserialize)]
struct MessageEnvelope {
    #[serde(rename = "type")]
    message_type: String,
    payload: serde_json::Value,
    timestamp: String,
}

async fn handle_message(msg: ws::Message, state: &AppState) -> GameResult<()> {
    let text = msg.into_text()?;
    let envelope: MessageEnvelope = serde_json::from_str(&text)?;
    
    match envelope.message_type.as_str() {
        "Authenticate" => {
            let payload: AuthPayload = serde_json::from_value(envelope.payload)?;
            handle_authenticate(payload, state).await
        }
        "PlayerAction" => {
            let payload: PlayerActionPayload = serde_json::from_value(envelope.payload)?;
            handle_player_action(payload, state).await
        }
        _ => Err(GameError::InvalidAction(format!("Unknown message type: {}", envelope.message_type)))
    }
}
```

**Message Handling Rules:**
- Deserialize into strongly-typed structs (no raw JSON manipulation)
- Match on `message_type` string, then deserialize specific payload
- Return structured errors for invalid messages
- Log received messages at DEBUG level
- Validate all fields after deserialization

### State Management Patterns

**Shared State with Arc<Mutex<>>:**

```rust
pub struct AppState {
    pub games: Arc<Mutex<HashMap<GameId, GameState>>>,
    pub sessions: Arc<Mutex<HashMap<SessionToken, PlayerId>>>,
    pub db: Arc<Mutex<Connection>>,
}

// In WebSocket handler
async fn handle_game_action(
    game_id: GameId,
    action: PlayerAction,
    state: web::Data<AppState>
) -> GameResult<()> {
    let mut games = state.games.lock().unwrap();
    let game = games.get_mut(&game_id)
        .ok_or_else(|| GameError::StateError("Game not found".into()))?;
    
    game.apply_action(action)?;
    
    // Release lock before async operations
    drop(games);
    
    // Sync critical state to database
    state.db.lock().unwrap().save_game_state(game_id, game)?;
    
    Ok(())
}
```

**State Management Rules:**
- Use `Arc<Mutex<>>` for shared mutable state across threads
- **Minimize lock duration:** Acquire, modify, release quickly
- **Avoid deadlocks:** Never acquire multiple locks in different orders
- **Clone data** if you need to hold it across async boundaries
- Sync critical state changes to database immediately

### Logging Conventions

**Log Level Usage:**

```rust
use tracing::{error, warn, info, debug, trace, instrument};

#[instrument(skip(state))]
async fn handle_player_action(
    player_id: u64,
    action: PlayerAction,
    state: &AppState
) -> GameResult<()> {
    info!(player_id, action_type = ?action, "Processing player action");
    
    // Validation
    if !is_valid_action(&action) {
        warn!(player_id, action_type = ?action, "Invalid player action");
        return Err(GameError::InvalidAction("Action not allowed".into()));
    }
    
    // Processing
    debug!(player_id, "Updating game state");
    match apply_action(player_id, action, state).await {
        Ok(_) => {
            info!(player_id, "Player action completed successfully");
            Ok(())
        }
        Err(e) => {
            error!(player_id, error = %e, "Failed to process player action");
            Err(e)
        }
    }
}
```

**Logging Rules:**
- **ERROR:** Unrecoverable errors, state corruption, critical failures
- **WARN:** Recoverable errors, invalid player actions, rate limits exceeded, security events
- **INFO:** Connection events, game state transitions (hand start/end), authentication
- **DEBUG:** Detailed message processing, game logic internals (development only)
- **TRACE:** Verbose debugging, every function call (rarely used)
- Use `#[instrument]` macro for automatic span tracking
- Include relevant context: player_id, game_id, action_type
- Log errors with `%e` for Display formatting

### Enforcement Guidelines

**All AI Agents MUST:**

1. **Use `rustfmt` and `clippy`** before committing code:
   ```bash
   cargo fmt --check
   cargo clippy -- -D warnings
   ```

2. **Return `Result<T, ErrorType>`** for all fallible operations
   - NO `unwrap()` or `expect()` in production code

3. **Use strongly-typed enums** for messages and game actions
   - Leverage Rust's type system for compile-time safety

4. **Follow module organization** as defined above
   - Game logic in `game/`, auth in `auth.rs`, etc.

5. **Write tests** for all public functions
   - Property-based tests for game logic
   - Integration tests for WebSocket flows

6. **Log at appropriate levels** following conventions above
   - Include context (player_id, game_id, etc.)

7. **Use `Arc<Mutex<>>` patterns** for shared mutable state
   - Minimize lock duration, avoid deadlocks

8. **Sequential migration numbering** starting at 001
   - Descriptive names, one logical change per migration

**Pattern Verification:**
- Pre-commit hooks run `cargo fmt -- check` and `cargo clippy`
- CI pipeline enforces all checks before merging
- Code review checklist includes pattern compliance

### Pattern Examples

**Good Example - Error Handling:**
```rust
pub fn evaluate_hand(cards: &[Card]) -> GameResult<HandRank> {
    if cards.len() < 5 {
        return Err(GameError::InvalidAction(
            format!("Need at least 5 cards, got {}", cards.len())
        ));
    }
    
    // ... evaluation logic ...
    Ok(HandRank::Flush)
}
```

**Anti-Pattern - Using unwrap():**
```rust
// DON'T DO THIS
pub fn evaluate_hand(cards: &[Card]) -> HandRank {
    assert!(cards.len() >= 5);  // Will panic in production!
    cards.get(0).unwrap()        // Will crash if cards empty!
    // ...
}
```

**Good Example - Structured Logging:**
```rust
#[instrument(skip(state))]
async fn join_table(player_id: u64, table_id: u64, state: &AppState) -> GameResult<()> {
    info!(player_id, table_id, "Player joining table");
    // ... logic ...
    info!(player_id, table_id, "Player successfully joined table");
    Ok(())
}
```

**Anti-Pattern - Unstructured Logging:**
```rust
async fn join_table(player_id: u64, table_id: u64, state: &AppState) -> GameResult<()> {
    println!("Player {} joining table {}", player_id, table_id);  // Not structured, no levels
    // ...
}
```

---

**Architecture Document Complete**

This architecture document provides comprehensive guidance for implementing the Headsup poker server, including:
- Technology stack and dependencies
- Core architectural decisions
- Implementation patterns and consistency rules

AI agents implementing this architecture should follow all documented patterns to ensure compatible, maintainable code.
