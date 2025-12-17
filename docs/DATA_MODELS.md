# DATA_MODELS.md

This document defines the **data structures and schemas** used throughout the system, including database schemas, API data structures, and session management.

---

## Related Documents

This document should be read **after**:

* README.md
* ARCHITECTURE.md
* PRODUCT_MODEL.md
* WORLD_FORMAT.md

---

## Principles

* Explicit schemas for all data
* Versioned data structures
* Immutable where possible
* Normalized for consistency
* Denormalized for performance where needed

---

## Database Schema

### Workspace

```sql
CREATE TABLE workspaces (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  billing_tier VARCHAR(50) NOT NULL, -- 'free', 'paid', 'enterprise'
  settings JSONB
);
```

### User

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  display_name VARCHAR(255) NOT NULL,
  password_hash VARCHAR(255), -- NULL if OAuth-only
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  last_seen_at TIMESTAMP,
  avatar_id UUID REFERENCES assets(id),
  preferences JSONB
);
```

### Workspace Membership

```sql
CREATE TABLE workspace_memberships (
  id UUID PRIMARY KEY,
  workspace_id UUID NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role VARCHAR(50) NOT NULL, -- 'Admin', 'Editor', 'Member', 'Guest'
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  UNIQUE(workspace_id, user_id)
);
```

### World

```sql
CREATE TABLE worlds (
  id UUID PRIMARY KEY,
  workspace_id UUID NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) NOT NULL,
  version VARCHAR(50) NOT NULL,
  world_data JSONB NOT NULL, -- WORLD_FORMAT.md compliant
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  created_by UUID REFERENCES users(id),
  UNIQUE(workspace_id, slug)
);
```

### Asset

```sql
CREATE TABLE assets (
  id UUID PRIMARY KEY,
  workspace_id UUID NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  type VARCHAR(50) NOT NULL, -- 'tileset', 'object', 'avatar', 'prop'
  storage_path VARCHAR(500) NOT NULL,
  metadata JSONB,
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL,
  created_by UUID REFERENCES users(id),
  is_custom BOOLEAN NOT NULL DEFAULT false,
  is_marketplace BOOLEAN NOT NULL DEFAULT false
);
```

### Session

```sql
CREATE TABLE sessions (
  id UUID PRIMARY KEY,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  workspace_id UUID REFERENCES workspaces(id),
  token_hash VARCHAR(255) NOT NULL,
  refresh_token_hash VARCHAR(255) NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  created_at TIMESTAMP NOT NULL,
  last_used_at TIMESTAMP NOT NULL,
  ip_address INET,
  user_agent VARCHAR(500)
);
```

### Audit Log

```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY,
  workspace_id UUID REFERENCES workspaces(id),
  user_id UUID REFERENCES users(id),
  action VARCHAR(100) NOT NULL,
  resource_type VARCHAR(50),
  resource_id UUID,
  details JSONB,
  ip_address INET,
  created_at TIMESTAMP NOT NULL
);
```

---

## API Data Structures

### User Object

```typescript
interface User {
  id: string; // UUID
  email: string;
  display_name: string;
  avatar_id?: string;
  workspace_id: string;
  roles: string[]; // ['Admin', 'Editor', 'Member']
  created_at: string; // ISO 8601
  last_seen_at?: string; // ISO 8601
}
```

### Workspace Object

```typescript
interface Workspace {
  id: string; // UUID
  name: string;
  slug: string;
  billing_tier: 'free' | 'paid' | 'enterprise';
  created_at: string; // ISO 8601
  updated_at: string; // ISO 8601
  settings: {
    max_worlds?: number;
    max_custom_assets?: number;
    max_users_per_world?: number;
  };
}
```

### World Object

```typescript
interface World {
  id: string; // UUID
  workspace_id: string;
  name: string;
  slug: string;
  version: string;
  created_at: string; // ISO 8601
  updated_at: string; // ISO 8601
  created_by: string; // User ID
  // Full world data follows WORLD_FORMAT.md structure
}
```

### World Summary (List View)

```typescript
interface WorldSummary {
  id: string;
  workspace_id: string;
  name: string;
  slug: string;
  version: string;
  created_at: string;
  updated_at: string;
  created_by: string;
  user_count?: number; // Current users in world
}
```

### Asset Object

```typescript
interface Asset {
  id: string; // UUID
  workspace_id: string;
  name: string;
  type: 'tileset' | 'object' | 'avatar' | 'prop';
  storage_path: string;
  url: string; // CDN URL
  metadata: {
    width?: number;
    height?: number;
    collision?: boolean;
    interactions?: string[];
    occupancy?: 'single' | 'multiple';
  };
  created_at: string;
  updated_at: string;
  created_by: string;
  is_custom: boolean;
  is_marketplace: boolean;
}
```

### Presence Object

```typescript
interface Presence {
  user_id: string;
  world_id: string;
  zone_id: string;
  position: {
    x: number;
    y: number;
  };
  facing: 'north' | 'south' | 'east' | 'west';
  state: 'idle' | 'walking' | 'sitting';
  avatar_id: string;
  last_update: number; // Unix timestamp
}
```

### Object Lock

```typescript
interface ObjectLock {
  object_id: string;
  user_id: string;
  lock_id: string; // UUID
  acquired_at: number; // Unix timestamp
  expires_at?: number; // Unix timestamp (optional timeout)
}
```

---

## Session Management

### Session Data (Server-side)

```typescript
interface Session {
  id: string; // UUID
  user_id: string;
  workspace_id?: string;
  token_hash: string;
  refresh_token_hash: string;
  expires_at: Date;
  created_at: Date;
  last_used_at: Date;
  ip_address?: string;
  user_agent?: string;
}
```

### Session State (Client-side)

```typescript
interface ClientSession {
  access_token: string; // JWT (memory only)
  refresh_token: string; // Stored securely
  user: User;
  workspace?: Workspace;
  current_world?: string;
  expires_at: number; // Unix timestamp
}
```

---

## World Data Structure

World data follows `WORLD_FORMAT.md`. Key structures:

### World File

```typescript
interface WorldFile {
  worldId: string;
  workspaceId: string;
  version: string;
  meta: {
    name: string;
    createdAt: string;
    updatedAt: string;
    createdBy: string;
  };
  map: MapData;
  zones: Zone[];
  doors: Door[];
  objects: Object[];
  spawnPoints: SpawnPoint[];
}
```

See `WORLD_FORMAT.md` for complete structure definitions.

---

## Asset Metadata

### Tileset Metadata

```json
{
  "width": 32,
  "height": 32,
  "tile_count": 256,
  "collision_map": [0, 1, 1, ...]
}
```

### Object Metadata

```json
{
  "width": 64,
  "height": 64,
  "interactions": ["sit", "use"],
  "occupancy": "single",
  "collision": true,
  "linked_objects": ["chair-1"]
}
```

### Avatar Metadata

```json
{
  "width": 48,
  "height": 48,
  "animations": ["idle", "walk", "sit"],
  "customizable": true
}
```

---

## Caching Strategy

### Cache Keys

* `workspace:{id}` - Workspace data
* `world:{id}` - World data
* `user:{id}` - User data
* `presence:{world_id}:{zone_id}` - Zone presence
* `asset:{id}` - Asset metadata

### Cache TTL

* Workspace: 1 hour
* World: 5 minutes (frequently updated)
* User: 15 minutes
* Presence: 5 seconds (ephemeral)
* Asset: 24 hours (immutable)

---

## Data Validation

### Input Validation

* All UUIDs validated for format
* All timestamps validated for ISO 8601
* All JSON validated against schemas
* All strings validated for length and content

### Output Validation

* All responses validated before sending
* Type checking at API boundaries
* Schema validation for database writes

---

## Data Migration

### Versioning

* Database schema versioned
* World format versioned (see WORLD_FORMAT.md)
* API versioned (see SIGNALING_PROTOCOL.md)

### Migration Strategy

* Forward-compatible changes preferred
* Breaking changes require version increment
* Migration scripts for database changes
* Backward compatibility maintained where possible

---

## Explicit Rules

> All data structures must be explicitly defined.

> Database schemas must match API structures.

> World data must conform to WORLD_FORMAT.md.

> All timestamps are UTC and ISO 8601 format.

> All UUIDs are RFC 4122 compliant.

---

## Final Note

These data models ensure:

* Consistency across components
* Type safety
* Validation at boundaries
* Clear data flow
* Easy debugging

Any data structure not defined here should be added to this document before implementation.

