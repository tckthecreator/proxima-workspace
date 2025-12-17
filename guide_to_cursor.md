Perfeito. Abaixo está o **mapeamento exato**, documento por documento, com **texto pronto para ser inserido**.
A ideia é: você pode literalmente copiar cada bloco e pedir ao Cursor para **inserir exatamente onde indicado**.

Vou respeitar o modelo final acordado:

* README.md (navegação + visão)
* PRODUCT_MODEL.md (o “o quê”)
* WORLD_FORMAT.md (o “como” estrutural)
* ARCHITECTURE.md (execução técnica)

Nada além desses quatro.

---

# 1. ARCHITECTURE.md

## ➕ Nova seção: *Movement & Authority Model*

**Inserir como seção própria.**

```md
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
```

---

## ➕ Nova seção: *Presence Synchronization*

```md
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
```

---

## ➕ Nota curta em *Moderation Path*

(Se já existir algo sobre moderação, **apenas acrescente**.)

```md
All moderation and administrative actions are executed via the Signaling Server and are not part of the spatial or ZoneType system.
```

---

# 2. PRODUCT_MODEL.md

## ➕ Nova seção: *Interactive Objects*

````md
## Interactive Objects

Objects in the World may declare explicit interaction and occupancy behavior.

### Occupancy Model

Each interactive object declares its occupancy policy:

```json
"occupancy": "single" | "multiple"
````

### Rules

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

### Explicit Rule

> All object locks are resolved by the Signaling Server.

````

---

## ➕ Nova seção: *Hard Limits & Constraints*

```md
## Hard Limits & Constraints

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
````

---

## ➕ Nova seção: *Moderation & Administrative Actions*

```md
## Moderation & Administrative Actions

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
```

---

# 3. WORLD_FORMAT.md

## ➕ Extensão em *Object Definition*

(Adicionar à definição de objetos interativos.)

````md
### Object Occupancy

Interactive objects may define an occupancy policy:

```json
{
  "id": "chair-42",
  "type": "Chair",
  "occupancy": "single"
}
````

### Semantics

* Objects with `single` occupancy require a server-granted lock.
* Objects with `multiple` occupancy do not require locking.
* Occupancy state is not inferred from animation.

````

---

## ➕ Nota curta sobre colisão

```md
Collision is evaluated independently of avatar customization.

Avatar visual customization must not affect:
- collision boundaries
- hitboxes
- interaction range
````

---

# 4. README.md

## ➕ Pequeno ajuste na seção de princípios

Adicionar ao bloco de princípios fundamentais:

```md
- Movement is client-optimistic and server-corrective
- Presence is synchronized by Zone, never by World
- Object interaction conflicts are resolved server-side
- Product limits are explicit and intentional
- Moderation is administrative, not spatial
```

---

# Checklist Final (para você validar com o Cursor)

* [ ] ARCHITECTURE.md: Movement & Presence adicionados
* [ ] PRODUCT_MODEL.md: Objects, Limits, Moderation adicionados
* [ ] WORLD_FORMAT.md: Occupancy formalizada
* [ ] README.md: princípios atualizados
* [ ] Nenhum novo documento criado
* [ ] Nenhuma regra duplicada em lugares errados

Se quiser, no próximo passo eu posso:

* revisar **após** o Cursor aplicar as mudanças, ou

Você está oficialmente no estágio de *design congelado*.
