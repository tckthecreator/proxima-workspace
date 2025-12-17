# ARCHITECTURE.md

This document defines the **technical architecture** of the platform and how all components interact at runtime.

It is the authoritative reference for infrastructure, networking, media flow and deployment assumptions.

---

## Related Documents

This document is designed to be read **after**:

* README.md
* PRODUCT_MODEL.md
* WORLD_FORMAT.md
* BUILD_MODE_AND_ASSETS.md

---

## Architectural Goals

* Predictable real-time behavior
* Progressive scalability (small → large worlds)
* Minimal operational complexity
* First-class self-hosting via Docker
* Clear separation of concerns

---

## High-Level Components

The system is composed of five primary components:

1. Client
2. Signaling Server
3. Media Plane (P2P / TURN / SFU)
4. Persistence Layer
5. Asset Delivery

Each component has a strict responsibility boundary.

---

## Client

The Client is a **stateful real-time application** executed by end users.

### Technology Stack

**Language:** TypeScript/JavaScript

**Recommended Technologies:**
* **Rendering:** PixiJS (2D/2.5D isometric rendering)
* **UI Framework:** React (for UI outside canvas)
* **State Management:** Zustand or similar (local state)
* **WebRTC:** Native WebRTC APIs (audio, video, data channels)
* **Audio:** Web Audio API (spatial audio, proximity-based)
* **Build Tool:** Vite or Webpack
* **Package Manager:** npm or yarn

**Rationale:**
* Web-based for zero-install deployment
* TypeScript for type safety and maintainability
* PixiJS for performant 2D rendering
* React for declarative UI
* Native WebRTC for peer-to-peer media

### Responsibilities

* Render the World (2.5D isometric)
* Enforce collision and spatial rules
* Execute ZoneType behavior
* Manage avatar state
* Establish WebRTC connections

The Client is authoritative over:

* Local movement
* Collision resolution
* Object interaction chains

The Client is **never authoritative** over:

* Presence
* Permissions
* Media topology decisions

---

## Movement & Authority Model

Movement in the World follows an optimistic client model with server-side correction.

### Principles

- The Client applies movement immediately for responsiveness.
- The Client sends a `MoveIntent` to the Signaling Server.
- The Signaling Server validates:
  - collisions (against map boundaries and walls)
  - exclusive occupancy (doors, chairs, narrow passages)
  - zone transitions

If the movement is invalid, the Signaling Server issues a `PositionCorrection`.

The Client must reconcile this correction smoothly (snap-back within 1–2 frames), never with a hard teleport unless strictly necessary.

### Authority Clarification

The phrase "corrective, never authoritative" means:

- **Client is authoritative for rendering**: Movement is applied immediately for responsive UX
- **Server is authoritative for validation**: Collision and occupancy rules are enforced server-side
- **Server does not drive movement**: The server never initiates movement, only validates and corrects client-initiated movement

This model guarantees:
- low-latency movement
- predictable corrections
- no server-authoritative movement pipeline (server doesn't push positions, only corrects invalid ones)

### Explicit Rule

> The Client is optimistic for UX, the Signaling Server is authoritative for validation.

---

## Presence Synchronization

Presence synchronization is delta-based and scoped strictly by Zone.

### Model

Clients periodically broadcast:
- position (x, y)
- facing direction
- avatar state (idle, walking, sitting)

Recommended update frequency:
- 10–15 Hz maximum per client

### Zone Determination

Zone membership is determined by spatial bounds:

1. **Client-side calculation**: The Client calculates which zone(s) contain the avatar's current position using zone bounds from the World file.
2. **Server-side validation**: The Signaling Server maintains authoritative zone membership and validates zone transitions.
3. **Overlap resolution**: When multiple zones overlap, the most specific ZoneType applies (PrivateRoom > MeetingRoom > TeamArea > OpenArea).
4. **Boundary handling**: If an avatar is exactly on a zone boundary, the zone with the highest priority applies. If zones have equal priority, the first zone in the World file's zone list applies.

When zone membership changes:
- Client sends `ZoneTransition` message to Signaling Server
- Server validates the transition and updates presence state
- Server broadcasts zone membership changes to affected clients
- Full presence snapshot is sent to the client for the new zone

### Scope Rules

- Presence updates are broadcast **only to Clients within the same Zone**.
- There is no continuous global World snapshot.
- Full presence snapshots occur only:
  - when entering a World
  - when transitioning between Zones

### Explicit Rule

> Presence is synchronized by Zone, never by World.

---

## Signaling Server

The Signaling Server is the **control plane**.

### Technology Stack

**Language:** Golang (Go)

**Recommended Technologies:**
* **Web Framework:** Gorilla WebSocket or similar
* **Database:** PostgreSQL (via pgx or GORM)
* **Caching:** Redis (optional, for presence caching)
* **Authentication:** JWT (via golang-jwt/jwt)
* **Configuration:** Viper or environment variables
* **Metrics:** Prometheus client library

**Rationale:**
* Golang for high concurrency and low latency
* Excellent WebSocket support
* Strong standard library
* Easy Docker deployment
* Good performance for real-time systems

### Responsibilities

* User presence
* World join/leave
* Zone enter/exit
* WebRTC signaling
* Policy decisions (P2P vs TURN vs SFU)
* Object lock management

Non-responsibilities:

* Media forwarding
* Rendering
* World mutation

The Signaling Server is stateless or lightly stateful and horizontally scalable.

All moderation and administrative actions are executed via the Signaling Server and are not part of the spatial or ZoneType system.

### Object Locking Protocol

For objects with `single` occupancy, the Signaling Server manages exclusive locks:

**Lock Acquisition Flow:**
1. Client sends `InteractRequest(objectId)` to Signaling Server
2. Server checks if object is currently locked
3. If available:
   - Server grants lock and sends `InteractGranted(objectId)`
   - Server broadcasts `ObjectOccupied(objectId, userId)` to zone participants
   - Client applies interaction (e.g., sit animation)
4. If locked:
   - Server sends `InteractDenied(objectId, reason: "occupied")`
   - Client displays feedback (visual shake, audio cue)
   - No forced teleport occurs

**Lock Release:**
- Lock is released when:
  - Avatar explicitly leaves the object (sends `InteractEnd(objectId)`)
  - Avatar moves away from object position (detected by server)
  - Client disconnects (automatic release)
  - Avatar transitions to different zone

**Simultaneous Requests:**
- If multiple clients request the same object simultaneously:
  - Server uses first-come-first-served (FCFS) based on message timestamp
  - First requestor receives lock
  - Other requestors receive denial
  - No queuing system (users must retry manually)

**Lock Persistence:**
- Locks are ephemeral and exist only while:
  - Client maintains connection to Signaling Server
  - Avatar remains in the same zone as the object
  - Object interaction is active

**Lock State:**
- Server maintains lock state in memory (not persisted)
- On server restart, all locks are released
- Clients detect lock loss via heartbeat timeout and release interaction locally

---

## Media Plane

Media transmission follows a **progressive escalation model**:

1. Direct P2P (WebRTC)
2. TURN relay (fallback)
3. SFU (Selective Forwarding Unit)

### Technology Stack

**TURN Server:**
* **Language:** Golang
* **Implementation:** pion/turn
* **Deployment:** Docker container (Go binary)
* **Configuration:** Go config file or environment variables
* **Rationale:** Single language stack simplifies development, deployment, and maintenance. pion/turn provides excellent performance and integrates seamlessly with Go-based infrastructure.

**SFU:**
* **Language:** Golang
* **Libraries:** pion/webrtc or livekit
* **Rationale:** Single language stack (Go) across all backend components. Go provides excellent performance for media forwarding with efficient memory management and strong concurrency support.

### P2P

Used when:

* Small number of peers
* Network conditions allow

Pros:

* Zero infrastructure cost
* Lowest latency

Cons:

* Poor scalability

---

### TURN

Used when:

* NAT traversal fails

Pros:

* Improves connectivity

Cons:

* High bandwidth cost
* Still N²

---

### SFU

Used when:

* Zone population exceeds threshold
* MeetingRoom is active

Responsibilities:

* Receive WebRTC streams
* Forward streams selectively

Non-responsibilities:

* Mixing
* Recording (unless explicitly enabled)

SFU is **self-hostable via Docker**.

---

## Media Policy Resolution

Decisions are made by the Signaling Server based on:

* ZoneType
* Participant count
* Network conditions

### Policy Thresholds

The following thresholds determine media topology:

**P2P (Direct)**
- Used when:
  - Zone population ≤ 8 participants
  - ZoneType is OpenArea or PrivateRoom
  - All participants have successful NAT traversal

**TURN (Relay)**
- Used when:
  - P2P direct fails for individual participants
  - Applied per-participant, not zone-wide
  - Falls back to P2P direct if TURN succeeds

**SFU (Selective Forwarding Unit)**
- Used when:
  - Zone population > 12 participants
  - ZoneType is MeetingRoom (always uses SFU)
  - ZoneType is TeamArea with > 8 participants
  - > 30% of participants require TURN relay
  - Average packet loss > 5% across zone
  - Average CPU usage > 70% across clients in zone

### Transition Logic

Media topology transitions occur automatically:
1. Signaling Server monitors zone metrics
2. When threshold is crossed, server initiates topology change
3. Clients receive topology change command
4. New connections are established in parallel
5. Old connections are torn down after new ones are stable
6. Transition occurs without dropping media (seamless handoff)

Clients never choose topology autonomously.

---

## Persistence Layer

Persistent data includes:

* World files
* Asset metadata
* Workspace configuration

Non-persistent data:

* Avatar position
* Ephemeral presence

### Technology Stack

**Database:**
* **Primary:** PostgreSQL
* **Rationale:** ACID compliance, JSONB support, strong ecosystem

**Object Storage:**
* **Options:** S3-compatible storage (MinIO, AWS S3, etc.)
* **Use Case:** World files, asset files (images, sprites)
* **Rationale:** Scalable, cost-effective for large files

**Caching (Optional):**
* **Redis:** For presence caching, session management
* **Rationale:** Low latency, high throughput

---

## Asset Delivery

Assets are delivered via:

* CDN or static file server

Rules:

* Immutable assets
* Cacheable
* Identified by stable IDs

---

## Deployment Model

All components are designed to be **self-hosted via Docker**.

### Backend Stack (Golang)

All backend components use **Golang** for a unified, single-language stack:

* **Signaling Server** - Go
* **TURN Server** - Go (pion/turn)
* **SFU** - Go (pion/webrtc or livekit)
* **Asset Server** - Go (or nginx for static files)

### Minimal Deployment

Minimal deployment includes:

* Signaling Server (Go)
* TURN (optional, Go)
* SFU (optional, Go)
* Asset server (Go or nginx)

**Benefits of Single-Language Stack:**
* Unified toolchain and dependencies
* Shared code and utilities
* Consistent deployment patterns
* Easier team onboarding
* Simplified debugging and observability

No component requires manual installation.

---

## Scalability Model

Scaling strategies:

* Horizontal scaling of Signaling Server
* Sharding by World
* Zone-based SFU allocation

No global shared state is required.

---

## Failure Isolation

Failures are isolated per plane:

* Client failures do not cascade
* Media failures degrade gracefully
* Signaling failures disconnect but do not corrupt state

For detailed failure handling, see FAILURE_MODES.md.

---

## Non-Goals

This architecture explicitly avoids:

* Server-authoritative movement
* Server-side rendering
* Hidden background jobs

---

## Final Note

The architecture is intentionally conservative.

Every added component must justify itself in terms of:

* Latency
* Operational cost
* Cognitive load

Simplicity is a feature.
