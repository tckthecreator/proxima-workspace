# WORLD_FORMAT.md

This document defines **how a World is represented on disk** and transmitted to Clients.

It is the **single source of truth for spatial data**, strictly implementing the rules defined in `PRODUCT_MODEL.md`.

---

## Related Documents

This document is designed to be read **after**:

* PRODUCT_MODEL.md (defines *what* exists and *why*)
* README.md (navigation and vocabulary)

---

## Design Principles

* Declarative over imperative
* Deterministic behavior (no runtime inference)
* Human-readable and diff-friendly
* Safe for version control
* Engine-agnostic, Client-driven

---

## Top-Level Structure

```json
{
  "worldId": "acme-backend-world",
  "workspaceId": "acme-corp",
  "version": "1.0.0",
  "meta": { ... },
  "map": { ... },
  "zones": [ ... ],
  "doors": [ ... ],
  "objects": [ ... ],
  "spawnPoints": [ ... ]
}
```

---

## Meta

```json
"meta": {
  "name": "Backend Department",
  "createdAt": "2025-01-10T12:00:00Z",
  "updatedAt": "2025-02-01T18:30:00Z",
  "createdBy": "admin@acme.com"
}
```

Purely informational. No runtime behavior.

---

## Map

Defines the **visual and collision layer**.

```json
"map": {
  "tileSet": "office-v1",
  "width": 120,
  "height": 80,
  "layers": [
    {
      "id": "floor",
      "tiles": [ ... ]
    },
    {
      "id": "walls",
      "tiles": [ ... ],
      "collision": true
    }
  ]
}
```

### Collision Rules

* Walls and blocked tiles are authoritative
* Avatars **cannot** pass through collidable tiles
* Client enforces collision locally

Collision is evaluated independently of avatar customization.

Avatar visual customization must not affect:
- collision boundaries
- hitboxes
- interaction range

---

## Zones

Zones define **behavioral context**.

```json
"zones": [
  {
    "id": "team-backend",
    "type": "TeamArea",
    "bounds": {
      "x": 10,
      "y": 20,
      "width": 40,
      "height": 25
    },
    "overrides": {
      "video": "optional"
    }
  }
]
```

### Zone Resolution Rules

* Zones are resolved purely by `bounds`
* No visual inference
* Only allowed overlap:

  * `PersonalRoom` inside `TeamArea`

---

## Doors

Doors control **access**, not behavior.

```json
"doors": [
  {
    "id": "backend-meeting-door",
    "bounds": {
      "x": 22,
      "y": 30,
      "width": 2,
      "height": 1
    },
    "leadsToZone": "backend-meeting-room",
    "access": {
      "type": "knock",
      "allowedRoles": ["Admin", "Member"]
    }
  }
]
```

### Door Rules

* Doors are the **only entry control mechanism**
* Zones never perform permission checks
* Doors may block movement

---

## Objects

Objects are interactive entities.

```json
"objects": [
  {
    "id": "desk-12",
    "type": "Desk",
    "position": { "x": 18, "y": 26 },
    "linkedObjects": ["chair-12", "computer-12"]
  },
  {
    "id": "chair-12",
    "type": "Chair",
    "position": { "x": 18, "y": 27 },
    "interaction": {
      "action": "sit"
    }
  },
  {
    "id": "computer-12",
    "type": "Computer",
    "interaction": {
      "action": "use",
      "requires": "chair-12"
    }
  }
]
```

### Object Occupancy

Interactive objects may define an occupancy policy:

```json
{
  "id": "chair-42",
  "type": "Chair",
  "occupancy": "single"
}
```

### Semantics

* Objects with `single` occupancy require a server-granted lock.
* Objects with `multiple` occupancy do not require locking.
* Occupancy state is not inferred from animation.

### Object Graph Rules

* Objects may reference each other
* Client resolves interaction chains
* Objects may trigger animations but never override Zone rules

---

## Spawn Points

```json
"spawnPoints": [
  {
    "id": "main-entrance",
    "position": { "x": 6, "y": 10 }
  }
]
```

Rules:

* Used on every World entry
* Avatar position is never persisted

---

## Validation Rules

A World file is invalid if:

* Zone overlap rules are violated
* Doors reference unknown zones
* Objects reference missing objects
* Bounds exceed map dimensions

---

## Non-Goals

This format does **not** define:

* Rendering engine details
* Networking protocol
* Media handling

Those are covered elsewhere.

---

## Final Note

This format is designed to:

* Be editable by Build Mode
* Be version-controlled
* Be safely extended without breaking Clients

Any runtime behavior not derivable from this file is considered a bug.
