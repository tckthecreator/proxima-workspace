# BUILD_MODE_AND_ASSETS.md

This document defines **how Worlds are created, edited and extended**, including Build Mode behavior, asset management and marketplace rules.

It operationalizes the data model defined in `WORLD_FORMAT.md` and the constraints defined in `PRODUCT_MODEL.md`.

---

## Related Documents

This document should be read **after**:

* README.md
* PRODUCT_MODEL.md
* WORLD_FORMAT.md

---

## Core Principles

* Build Mode uses the **same rendering engine** as the Client
* Zero-install: entirely in-browser
* What you see is what gets saved (WYSIWYG)
* No hidden inference or implicit behavior
* Build Mode produces **only valid World files**

---

## Build Mode Overview

Build Mode is a **privileged Client state** that allows authorized users to modify a World.

Characteristics:

* Same Client binary
* Different permission set
* Deterministic output (`WORLD_FORMAT.md` compliant)

Build Mode is **not** a separate application.

---

## Entering Build Mode

Conditions:

* User role: `Admin` or `Editor`
* Explicit action (button / command)

Effects:

* World is locked for editing (single editor at a time)
* Presence remains visible
* Non-editing users are read-only

---

## Editing Capabilities

### Map Editing

Allowed:

* Place floor tiles
* Place walls
* Remove tiles

Rules:

* Collision tiles are explicit
* Walls always imply collision
* Editor must prevent invalid geometry

---

### Zone Editing

Capabilities:

* Create Zone by drawing bounds
* Assign `ZoneType`
* Configure allowed overrides

Rules:

* Zone bounds must align to tile grid
* Overlap rules enforced at edit-time
* No runtime mutation

---

### Door Editing

Capabilities:

* Place Door bounds
* Assign access policy
* Link to target Zone

Rules:

* Doors are mandatory for restricted access
* Doors may block movement
* Door bounds must overlap collidable tiles

---

### Object Placement

Capabilities:

* Place objects
* Rotate (if supported by asset)
* Link objects together

Rules:

* Object type defines allowed interactions
* Linking creates interaction graph
* Invalid references are forbidden

---

## Asset System

Assets are immutable resources referenced by Worlds.

Types:

* Tilesets
* Objects (chairs, desks, computers)
* Decorative props
* Avatars

Assets are identified by stable IDs.

---

## Asset Sources

### First-Party Assets

* Shipped with the platform
* Guaranteed compatibility
* Default option for MVP

---

### Marketplace Assets

* Created by third parties or internal team
* Purchased or licensed per Workspace
* Versioned

Rules:

* Assets cannot execute code
* Assets cannot modify rules

---

### Custom Uploaded Assets

Allowed:

* Images
* Sprite sheets

Restrictions:

* Size limits enforced
* Format whitelist
* Manual or automated moderation

Custom assets are **paid-only**.

---

## Asset Validation

Before an asset is accepted:

* Dimensions are validated
* Collision metadata is explicit
* Interaction capabilities are declared

Assets without metadata cannot be used in Build Mode.

---

## Saving & Validation

Before a World can be saved:

* Full schema validation
* Zone overlap validation
* Door integrity validation
* Object graph validation

Invalid Worlds **cannot be persisted**.

---

## Versioning & Publishing

* World version increments automatically
* No rollback support (by design)
* Previous versions may be retained for audit

Publishing is explicit.

---

## Non-Goals

Build Mode does **not**:

* Edit signaling logic
* Configure media servers
* Override ZoneType rules

---

## Final Note

Build Mode is intentionally conservative.

If something cannot be represented safely in `WORLD_FORMAT.md`, it must not be editable.

Ease of use never overrides determinism or correctness.
