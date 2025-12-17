# CLIENT_STATE_MACHINE.md

This document describes the **Client state machine** responsible for controlling presence, media, collaboration, and connectivity. It translates ZoneTypes and rules into **executable behavior**, ensuring predictability, performance, and resilience.

---

## Related Documents

This document should be read **after**:

* README.md
* ARCHITECTURE.md
* PRODUCT_MODEL.md
* SIGNALING_PROTOCOL.md

---

## Objectives

* Define clear user states
* Standardize transitions between ZoneTypes
* Control media start/stop
* Integrate P2P / TURN / SFU policy
* Prevent resource leaks
* Ensure graceful degradation

---

## Principles

* One active state per user
* Explicit and atomic transitions
* Media only exists when state allows
* Failures return to safe states
* State changes are deterministic

---

## Main States

### 1. Disconnected

Initial state or after critical failure.

**Characteristics:**

* No signaling connection
* No media
* Minimal UI
* Local state may be preserved

**Transitions:**

* `connect()` → Connecting

**Actions on Entry:**

* Clear all media tracks
* Release all object locks
* Reset presence state

---

### 2. Connecting

Establishing connection with Signaling Server.

**Characteristics:**

* WebSocket connection in progress
* Authentication in progress
* No media active

**Transitions:**

* `connection_success()` → Idle
* `connection_failure()` → Disconnected
* `timeout()` → Disconnected

**Actions:**

* Authenticate with token
* Establish WebSocket connection
* Request initial world state

**Timeout:**

* 10 seconds maximum
* Exponential backoff on retry

---

### 3. Idle

User connected, no active media.

**Characteristics:**

* Signaling connection active
* Presence active
* Movement in world allowed
* No WebRTC connections
* No media tracks

**Transitions:**

* `enterZone(OpenArea)` → OpenArea
* `enterZone(TeamArea)` → TeamArea
* `enterZone(MeetingRoom)` → MeetingRoom
* `enterZone(PrivateRoom)` → PrivateRoom
* `disconnect()` → Disconnected

**Actions:**

* Broadcast presence updates (10-15 Hz)
* Handle movement intents
* Process object interactions

---

### 4. OpenArea

Circulation with proximity-based audio.

**Characteristics:**

* Audio via P2P (proximity-based)
* Video disabled
* Screen share disabled
* Whiteboard unavailable

**Transitions:**

* `enterZone(TeamArea)` → TeamArea
* `enterZone(MeetingRoom)` → MeetingRoom
* `enterZone(PrivateRoom)` → PrivateRoom
* `leaveZone()` → Idle
* `disconnect()` → Disconnected

**Actions:**

* Initialize P2P audio connections
* Update distance-based audio attenuation
* Monitor zone population for SFU promotion

**Media Topology:**

* P2P_DIRECT (default)
* P2P_TURN (fallback per peer)
* Never SFU

---

### 5. TeamArea

Collaborative team area.

**Characteristics:**

* Group audio
* Video optional
* Screen share allowed
* Whiteboard available
* Integrations active

**Transitions:**

* `enterZone(MeetingRoom)` → MeetingRoom
* `enterZone(PrivateRoom)` → PrivateRoom
* `leaveZone()` → Idle
* `disconnect()` → Disconnected

**Actions:**

* Initialize group audio
* Negotiate SFU if population > 8
* Activate whiteboard
* Enable integrations

**Media Topology:**

* P2P_DIRECT (≤ 8 participants)
* SFU (recommended, > 8 participants)

---

### 6. MeetingRoom

Structured meeting space.

**Characteristics:**

* Audio active on entry
* Video optional
* Screen share allowed
* Whiteboard available
* SFU required

**Transitions:**

* `leaveZone()` → Idle
* `disconnect()` → Disconnected

**Actions:**

* Connect via SFU immediately
* Activate audio
* Enable all collaboration tools

**Media Topology:**

* SFU (always)

---

### 7. PrivateRoom

Individual controlled space.

**Characteristics:**

* Audio only between authorized participants
* Video optional
* Screen share allowed
* Whiteboard optional
* Entry requires permission

**Transitions:**

* `inviteUser()` (remains in PrivateRoom)
* `leaveZone()` → Idle
* `disconnect()` → Disconnected

**Actions:**

* Open audio for authorized participants
* Manage entry permissions
* Handle knock requests

**Media Topology:**

* P2P_DIRECT (default)
* SFU (optional, if > 8 participants)

---

## Media Sub-States

Media states are **orthogonal** to main states.

### AudioState

* `OFF` - No audio track
* `P2P` - P2P audio connection
* `SFU` - SFU audio connection

### VideoState

* `OFF` - No video track
* `CAMERA` - Camera video active
* `SCREEN_SHARE` - Screen sharing active

### CollaborationState

* `NONE` - No collaboration tools active
* `WHITEBOARD` - Whiteboard active
* `SCREEN_SHARE` - Screen share active

---

## Zone Transition Flow

### Detection

1. Client calculates zone membership from position
2. Client detects zone change
3. Client sends `ZONE_TRANSITION` to Signaling Server
4. Server validates transition
5. Server sends `ZONE_TRANSITIONED` with new zone state

### Media Handling During Transition

1. **Before leaving zone:**
   * Close media connections for old zone
   * Release object locks
   * Send `INTERACT_END` for all active objects

2. **During transition:**
   * No media active
   * Presence updates continue

3. **After entering zone:**
   * Receive presence snapshot
   * Initialize media per ZoneType rules
   * Establish new connections

### Grace Period

* 500ms grace period for zone boundaries
* Prevents rapid oscillation
* Client uses last valid zone during grace period

---

## Connectivity Policy

Connectivity decisions occur at two moments:

1. **Zone entry** - Initial topology based on ZoneType and population
2. **Dynamic change** - Promotion/demotion based on metrics

### Criteria

* ZoneType (MeetingRoom always SFU)
* Participant count (thresholds defined in ARCHITECTURE.md)
* ICE failures (> 30% triggers SFU)
* CPU/bandwidth metrics (> 70% CPU triggers SFU)
* Packet loss (> 5% triggers SFU)

### Transition Logic

1. Server monitors zone metrics
2. Server sends topology change command
3. Client establishes new connections in parallel
4. Client tears down old connections after new ones stable
5. Seamless handoff (no media drop)

---

## Failure Handling

### Media Failure

* **Symptom**: Track ended unexpectedly, ICE failed
* **Action**: Close affected track only
* **State**: Remain in current state
* **Recovery**: Automatic renegotiation

### SFU Failure

* **Symptom**: SFU disconnected, streams missing
* **Action**: Fallback to P2P (if allowed by ZoneType)
* **State**: Remain in current state
* **Recovery**: Reconnect to SFU when available

### Signaling Failure

* **Symptom**: WebSocket closed, timeout
* **Action**: Close all media, enter Connecting state
* **State**: Connecting → (reconnect) → Idle
* **Recovery**: Exponential backoff reconnection

### Critical Failure

* **Symptom**: Client crash, excessive CPU
* **Action**: Enter Disconnected state
* **State**: Disconnected
* **Recovery**: User-initiated reconnect

---

## State Persistence

### Persistent

* User preferences
* World selection
* Avatar selection

### Non-Persistent

* Current position
* Active zone
* Media state
* Object locks

---

## Observability

Client must emit events:

* `state_change` - State transition occurred
* `media_started` - Media track started
* `media_stopped` - Media track stopped
* `connectivity_changed` - Topology changed
* `zone_transitioned` - Zone changed
* `error` - Error occurred

These events feed logs and metrics.

---

## Explicit Rules

> The Client state machine is the **behavioral core** of the Client.

> All state transitions must be explicit and logged.

> Media state must match main state requirements.

> Failures must return to safe states (Idle or Disconnected).

---

## Final Note

This state machine ensures:

* Predictable behavior
* Resource efficiency
* Graceful degradation
* Clear error recovery

Any feature that affects state must:

* Declare which states it affects
* Define valid transitions
* Respect ZoneType rules
* Handle failures gracefully

