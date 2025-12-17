# PRODUCT_MODEL

## Purpose

This document defines **what the product is**, independent of technical implementation. It establishes the mental model, constraints, and core behaviors that guide all other documents.

This is the primary source of truth for product decisions.

**This document relates to:**

* WORLD_FORMAT.md
* BUILD_MODE_AND_ASSETS.md
* ARCHITECTURE.md

---

## 1. Product Definition

**Proxima Workspace** is a presence-driven virtual office designed to replicate the spatial, social, and functional dynamics of a real workplace.

It balances:

* Presence and informal interaction
* Structured collaboration
* Productivity and enterprise reliability

The product is intentionally positioned as a **work-first environment with social realism**, not a game.

---

## 2. Core Concepts

### 2.1 Workspace

A **Workspace** represents an organization.

Characteristics:

* Contains users, roles, permissions
* Owns Worlds, assets, and configuration
* Billing and identity boundary

---

### 2.2 World

A **World** is a persistent virtual environment inside a Workspace.

Rules:

* A Workspace may contain multiple Worlds
* Each World contains exactly **one Map**
* World state is persistent, except avatar positions

Examples:

* Headquarters
* Engineering Building
* Regional Office

---

### 2.3 Map

A **Map** defines the spatial layout of a World.

Properties:

* Single continuous space
* Tile-based, 2.5D isometric
* Contains Zones, Objects, Doors, and Spawn Points

There is no concept of map transitions inside a World.

---

## 3. Zones

### 3.1 ZoneType

A **Zone** is a rectangular or polygonal spatial area with an explicit **ZoneType**.

ZoneType is:

* Explicitly assigned
* Immutable at runtime
* The absolute authority for behavior

There is no automatic inference of ZoneType.

---

### 3.2 Supported ZoneTypes

Initial ZoneTypes:

* OpenArea
* TeamArea
* MeetingRoom
* PrivateRoom

---

### 3.3 Zone Composition Rules

* Zones may not arbitrarily overlap
* **PrivateRoom may exist inside TeamArea**
* No other overlap is permitted

If overlap exists, the most specific ZoneType applies (PrivateRoom > MeetingRoom > TeamArea > OpenArea).

---

## 4. Spatial Rules & Access Control

### 4.1 Collision and Boundaries

* Walls are hard collision boundaries
* Avatars cannot pass through walls

When a Client attempts to cross a blocked boundary:

* Movement is rejected by the engine
* A micro lateral slide occurs
* Optional feedback:

  * Soft bump animation
  * Subtle collision sound

Avatar customization **does not affect**:

* Collision
* Hitboxes
* Interaction range

---

### 4.2 Doors vs Zones (Conceptual Separation)

**Zones define behavior.**
**Doors define access.**

They are intentionally separate concepts.

* A Zone defines *what happens* inside
* A Door defines *who may enter* and *how*

A Zone may exist without Doors (open areas).
A Door always belongs to a physical object.

---

### 4.3 Door Rules

Doors:

* Are physical objects placed on the Map
* May be locked, unlocked, or conditional

Supported behaviors:

* Open access
* Role-based access
* Knock-to-request-entry

---

### 4.4 PrivateRoom Access Flow

For PrivateRoom Zones:

1. Door is closed by default
2. Unauthorized avatars cannot cross the threshold
3. User may initiate a **knock** interaction
4. Inside users receive a prompt
5. Entry is explicitly granted or denied

This interaction is mandatory and cannot be bypassed.

---

## 5. Avatars

### 5.1 Avatar Scope

* Avatars are **Workspace-specific**
* No global avatars across Workspaces

---

### 5.2 Avatar Creation Model

Supported:

* Preset avatars with customization
* Marketplace avatars (paid)
* User-uploaded avatars (paid, moderated)

---

### 5.3 Functional Constraints

Avatar customization is **purely aesthetic**.

It does not modify:

* Movement speed
* Collision shape
* Interaction distance

---

## 6. Objects & Interactions

### 6.1 Interactive Objects

Objects may define interaction capabilities.

Examples:

* Chair → sit
* Sofa → sit
* Desk → attach objects
* Computer → work interaction

Objects in the World may declare explicit interaction and occupancy behavior.

#### Occupancy Model

Each interactive object declares its occupancy policy:

```json
"occupancy": "single" | "multiple"
```

#### Rules

* `single` occupancy:

  * Only one avatar may occupy the object at a time.
  * Lock acquisition is resolved by the Signaling Server.
  * Lock is released when the avatar leaves the object.
* `multiple` occupancy:

  * No locking.
  * Shared animation only.

If an avatar fails to acquire a lock:

* No forced teleport occurs.
* The Client displays feedback:

  * subtle visual shake or highlight
  * short audio cue

#### Explicit Rule

> All object locks are resolved by the Signaling Server.

---

### 6.3 Object Graph & Affordances

Objects may form logical graphs:

* Computer is attached to a Desk
* Desk has one or more Chairs

When a user interacts with a Computer:

1. Engine resolves linked Chair
2. Avatar moves to Chair
3. Sit animation is triggered

If no valid Chair exists, interaction is denied.

---

## 7. Hard Limits & Constraints

The product enforces explicit limits as a deliberate design decision.

Limits are part of the product model, not emergent engine behavior.

### Initial Limits (Subject to Revision)

- Maximum users per World: 150
- Maximum users per Zone:
  - OpenArea: 50
  - TeamArea: 25
  - MeetingRoom: 16
- Maximum objects per Map: 2,000
- Maximum custom assets per Workspace:
  - Free tier: 0
  - Paid tiers: 100

### Enforcement

- Limits are validated in:
  - Client runtime
  - Build Mode
- Violations are prevented, not degraded.

### Explicit Rule

> Limits are enforced intentionally, not emergent.

---

## 8. Moderation & Administrative Actions

Moderation is handled as an administrative capability, independent of spatial or ZoneType rules.

### Capabilities

| Action             | Allowed Roles     |
|--------------------|-------------------|
| Mute user          | Admin, Editor     |
| Remove from Zone   | Admin             |
| Kick from World    | Admin             |
| Lock Door          | Admin             |

### Principles

- Moderation actions are not ZoneTypes.
- Moderation does not alter spatial rules.
- All actions are executed via the Signaling Server.

### Explicit Rule

> Moderation is orthogonal to spatial rules.

---

## 9. Product Boundaries

Explicit non-goals:

* No physics-based gameplay
* No combat or competitive mechanics
* No avatar stat systems
* No emergent rule inference

All behavior must be:

* Predictable
* Explicit
* Configurable by design

---

## 10. Design Principles

* Presence over abstraction
* Explicit rules over magic behavior
* Visual clarity over realism
* Predictability over flexibility

---

## 11. Source of Truth

If a behavior is not defined in this document, it is **not a product requirement**.

Implementation documents must conform to this model.
