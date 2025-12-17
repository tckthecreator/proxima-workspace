# Proxima Workspace — Presence-Driven Virtual Office

Proxima Workspace is a presence-driven virtual office focused on spatial interaction, lightweight performance, and self-hosted deployment. The product emphasizes *being somewhere together* rather than simulating a full 3D world.

This repository’s documentation follows a **strict source-of-truth policy** to avoid conceptual drift and over-documentation.

---

## Source of Truth

Only the documents listed below are considered authoritative. All design, implementation, and product decisions must be derived from them.

1. **PRODUCT_MODEL.md**
   Defines *what the product is*: concepts, invariants, user-facing behavior, and product rules.

2. **ARCHITECTURE.md**
   Defines *how the system works*: runtime components, networking, scalability, and deployment model.

3. **WORLD_FORMAT.md**
   Defines *how the world is described*: spatial data model, zones, objects, doors, and validation rules.

4. **BUILD_MODE_AND_ASSETS.md**
   Defines *how worlds are created and customized*: in-browser editor (Build Mode), asset system, and content pipeline.

No other documents should be treated as sources of truth.

---

## Recommended Reading Order

For first-time readers:

1. PRODUCT_MODEL.md
2. WORLD_FORMAT.md
3. BUILD_MODE_AND_ASSETS.md
4. ARCHITECTURE.md

This order mirrors the product’s mental model: **what → how it’s represented → how it’s built → how it runs**.

---

## Frozen Vocabulary

To maintain consistency across documentation and code, the following terms are frozen:

* **ZoneType** — always referred to as `ZoneType`
* **Client** — never “frontend” or “app”
* **Signaling Server** — single canonical name
* **Build Mode** — editor mode inside the Client
* **World** — the spatial environment loaded by the Client
* **Office** — a business-owned World instance

Avoid introducing synonyms for these terms.

---

## Core Principles

* 2D / 2.5D spatial world (not full 3D)
* Presence-first UX
* Low bandwidth and CPU usage
* In-browser editing (no native tools)
* Docker-first and self-host friendly
* Deterministic collision and interaction rules
* Avatar customization never affects collision or interaction ranges
* Movement is client-optimistic and server-corrective
* Presence is synchronized by Zone, never by World
* Object interaction conflicts are resolved server-side
* Product limits are explicit and intentional
* Moderation is administrative, not spatial

---

## Non-Goals

Proxima Workspace explicitly does **not** aim to be:

* A full 3D metaverse
* A game engine
* A photorealistic environment
* An art-heavy or hardware-intensive platform

---

## Contribution Guidance

When proposing changes:

* Identify which source-of-truth document is affected
* Update that document first
* Avoid creating new standalone concept documents

If a concept does not clearly belong to one of the four documents, it likely needs refinement.

---

## License

TBD
