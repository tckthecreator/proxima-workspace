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

Responsibilities:

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
  - collisions
  - exclusive occupancy (doors, chairs, narrow passages)

If the movement is invalid, the Signaling Server issues a `PositionCorrection`.

The Client must reconcile this correction smoothly (snap-back within 1–2 frames), never with a hard teleport unless strictly necessary.

### Explicit Rule

> The Client is optimistic, the Signaling Server is corrective, never authoritative.

This model guarantees:
- low-latency movement
- predictable corrections
- no server-authoritative movement pipeline

---

## Presence Synchronization

Presence synchronization is delta-based and scoped strictly by Zone.

### Model

Clients periodically broadcast:
- position (x, y)
- facing direction
- avatar state (idle, walking, sitting)

Recommended update frequency:
- 10–15 Hz maximum

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

Responsibilities:

* User presence
* World join/leave
* Zone enter/exit
* WebRTC signaling
* Policy decisions (P2P vs TURN vs SFU)

Non-responsibilities:

* Media forwarding
* Rendering
* World mutation

The Signaling Server is stateless or lightly stateful and horizontally scalable.

Recommended implementation: Golang.

All moderation and administrative actions are executed via the Signaling Server and are not part of the spatial or ZoneType system.

---

## Media Plane

Media transmission follows a **progressive escalation model**:

1. Direct P2P (WebRTC)
2. TURN relay (fallback)
3. SFU (Selective Forwarding Unit)

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

Storage options:

* Object storage (S3-compatible)
* Relational DB for metadata

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

Minimal deployment includes:

* Signaling Server
* TURN (optional)
* SFU (optional)
* Asset server

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

Refer to FAILURE_MODES.md for details.

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
