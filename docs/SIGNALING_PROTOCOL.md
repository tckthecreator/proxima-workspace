# SIGNALING_PROTOCOL.md

This document defines the **signaling protocol** used to coordinate WebRTC connections, zone states, and transitions between P2P, TURN, and SFU.

The Signaling Server **does not transmit media**. It only coordinates peers and topology decisions.

This protocol is **normative**: Client and server must implement it as described.

---

## Related Documents

This document should be read **after**:

* README.md
* ARCHITECTURE.md
* PRODUCT_MODEL.md

---

## 1. Signaling Objectives

* Peer discovery
* SDP and ICE candidate exchange
* ZoneType coordination
* Automatic promotion to SFU
* Minimal state synchronization
* Presence synchronization
* Object interaction coordination

---

## 2. Transport

* **WebSocket (JSON)**
* One connection per client
* Ephemeral state (no persistence)
* Connection is authenticated via token (see SECURITY_MODEL.md)

---

## 3. Message Envelope

All messages follow this envelope:

```json
{
  "type": "string",
  "payload": {},
  "protocol_version": 1
}
```

The `protocol_version` field allows clients and servers to negotiate compatibility.

---

## 4. Identities

### 4.1 User

```json
{
  "user_id": "uuid",
  "workspace_id": "uuid",
  "display_name": "string",
  "avatar_id": "string"
}
```

### 4.2 Zone

```json
{
  "zone_id": "uuid",
  "zone_type": "OpenArea | TeamArea | MeetingRoom | PrivateRoom",
  "world_id": "uuid"
}
```

---

## 5. Connection States

### 5.1 ClientConnectionState

* `CONNECTING` - Establishing WebSocket connection
* `CONNECTED` - Authenticated and ready
* `DISCONNECTED` - Connection lost or closed

### 5.2 MediaTopology

* `P2P_DIRECT` - Direct peer-to-peer
* `P2P_TURN` - P2P with TURN relay
* `SFU` - Selective Forwarding Unit

---

## 6. Initial Connection Flow

### 6.1 Connect

Client → Server

```json
{
  "type": "CONNECT",
  "payload": {
    "token": "jwt_token",
    "client_version": "1.0.0"
  },
  "protocol_version": 1
}
```

Server → Client

```json
{
  "type": "CONNECTED",
  "payload": {
    "user": { /* User */ },
    "server_version": "1.0.0"
  }
}
```

### 6.2 Join World

Client → Server

```json
{
  "type": "JOIN_WORLD",
  "payload": {
    "world_id": "uuid"
  }
}
```

Server → Client

```json
{
  "type": "WORLD_STATE",
  "payload": {
    "world_id": "uuid",
    "current_zone": { /* Zone */ },
    "media_topology": "P2P_DIRECT",
    "peers": [ /* User[] */ ]
  }
}
```

---

## 7. Presence Synchronization

### 7.1 Presence Update

Client → Server (10-15 Hz)

```json
{
  "type": "PRESENCE_UPDATE",
  "payload": {
    "position": { "x": 18, "y": 26 },
    "facing": "north",
    "state": "idle | walking | sitting",
    "zone_id": "uuid"
  }
}
```

### 7.2 Presence Broadcast

Server → Clients in same Zone

```json
{
  "type": "PRESENCE_BROADCAST",
  "payload": {
    "user_id": "uuid",
    "position": { "x": 18, "y": 26 },
    "facing": "north",
    "state": "idle | walking | sitting"
  }
}
```

### 7.3 Zone Transition

Client → Server

```json
{
  "type": "ZONE_TRANSITION",
  "payload": {
    "from_zone_id": "uuid",
    "to_zone_id": "uuid",
    "position": { "x": 20, "y": 30 }
  }
}
```

Server → Client

```json
{
  "type": "ZONE_TRANSITIONED",
  "payload": {
    "zone": { /* Zone */ },
    "presence_snapshot": [ /* User[] with positions */ ],
    "media_topology": "P2P_DIRECT | SFU"
  }
}
```

---

## 8. Movement & Collision

### 8.1 Move Intent

Client → Server

```json
{
  "type": "MOVE_INTENT",
  "payload": {
    "position": { "x": 19, "y": 27 },
    "timestamp": 1234567890
  }
}
```

### 8.2 Position Correction

Server → Client (if movement invalid)

```json
{
  "type": "POSITION_CORRECTION",
  "payload": {
    "correct_position": { "x": 18, "y": 26 },
    "reason": "collision | occupied | invalid"
  }
}
```

---

## 9. Object Interactions

### 9.1 Interact Request

Client → Server

```json
{
  "type": "INTERACT_REQUEST",
  "payload": {
    "object_id": "uuid",
    "action": "sit | use | attach"
  }
}
```

### 9.2 Interact Granted

Server → Client

```json
{
  "type": "INTERACT_GRANTED",
  "payload": {
    "object_id": "uuid",
    "lock_id": "uuid"
  }
}
```

### 9.3 Interact Denied

Server → Client

```json
{
  "type": "INTERACT_DENIED",
  "payload": {
    "object_id": "uuid",
    "reason": "occupied | invalid | unauthorized"
  }
}
```

### 9.4 Object Occupied Broadcast

Server → Clients in Zone

```json
{
  "type": "OBJECT_OCCUPIED",
  "payload": {
    "object_id": "uuid",
    "user_id": "uuid"
  }
}
```

### 9.5 Interact End

Client → Server

```json
{
  "type": "INTERACT_END",
  "payload": {
    "object_id": "uuid",
    "lock_id": "uuid"
  }
}
```

---

## 10. WebRTC Negotiation (P2P)

### 10.1 SDP Offer

Client → Server

```json
{
  "type": "SDP_OFFER",
  "payload": {
    "to": "user_id",
    "sdp": "base64_encoded_sdp"
  }
}
```

Server → Target Client

```json
{
  "type": "SDP_OFFER",
  "payload": {
    "from": "user_id",
    "sdp": "base64_encoded_sdp"
  }
}
```

### 10.2 SDP Answer

Client → Server

```json
{
  "type": "SDP_ANSWER",
  "payload": {
    "to": "user_id",
    "sdp": "base64_encoded_sdp"
  }
}
```

Server → Target Client

```json
{
  "type": "SDP_ANSWER",
  "payload": {
    "from": "user_id",
    "sdp": "base64_encoded_sdp"
  }
}
```

### 10.3 ICE Candidate

Client → Server

```json
{
  "type": "ICE_CANDIDATE",
  "payload": {
    "to": "user_id",
    "candidate": { /* ICE candidate object */ }
  }
}
```

Server → Target Client

```json
{
  "type": "ICE_CANDIDATE",
  "payload": {
    "from": "user_id",
    "candidate": { /* ICE candidate object */ }
  }
}
```

---

## 11. TURN Fallback

### 11.1 ICE Failed

Client → Server

```json
{
  "type": "ICE_FAILED",
  "payload": {
    "peer_id": "user_id"
  }
}
```

### 11.2 TURN Credentials

Server → Client

```json
{
  "type": "TURN_CREDENTIALS",
  "payload": {
    "urls": ["turn:turn.example.com:3478"],
    "username": "token",
    "credential": "token"
  }
}
```

---

## 12. SFU Promotion

### 12.1 Promote to SFU

Server → All Clients in Zone

```json
{
  "type": "PROMOTE_TO_SFU",
  "payload": {
    "sfu_url": "wss://sfu.example.com",
    "sfu_token": "jwt_token",
    "zone_id": "uuid"
  }
}
```

### 12.2 SFU Connected

Client → Server

```json
{
  "type": "SFU_CONNECTED",
  "payload": {
    "zone_id": "uuid"
  }
}
```

### 12.3 SFU Disconnected

Client → Server

```json
{
  "type": "SFU_DISCONNECTED",
  "payload": {
    "zone_id": "uuid",
    "reason": "error | manual"
  }
}
```

---

## 13. Peer Discovery

### 13.1 Peer Joined

Server → Clients in Zone

```json
{
  "type": "PEER_JOINED",
  "payload": {
    "user": { /* User */ },
    "position": { "x": 10, "y": 15 }
  }
}
```

### 13.2 Peer Left

Server → Clients in Zone

```json
{
  "type": "PEER_LEFT",
  "payload": {
    "user_id": "uuid"
  }
}
```

---

## 14. Errors

### 14.1 Error Message

Server → Client (or Client → Server)

```json
{
  "type": "ERROR",
  "payload": {
    "code": "AUTH_FAILED | INVALID_REQUEST | RATE_LIMITED | SERVER_ERROR",
    "message": "Human-readable error message",
    "details": {}
  }
}
```

### 14.2 Error Codes

* `AUTH_FAILED` - Authentication token invalid or expired
* `INVALID_REQUEST` - Malformed or invalid message
* `RATE_LIMITED` - Too many requests
* `SERVER_ERROR` - Internal server error
* `WORLD_NOT_FOUND` - World does not exist
* `ZONE_NOT_FOUND` - Zone does not exist
* `OBJECT_NOT_FOUND` - Object does not exist
* `PERMISSION_DENIED` - User lacks required permissions

---

## 15. Moderation Actions

### 15.1 Mute User

Admin/Editor → Server

```json
{
  "type": "MUTE_USER",
  "payload": {
    "target_user_id": "uuid",
    "duration": 300
  }
}
```

### 15.2 Remove from Zone

Admin → Server

```json
{
  "type": "REMOVE_FROM_ZONE",
  "payload": {
    "target_user_id": "uuid",
    "zone_id": "uuid"
  }
}
```

### 15.3 Kick from World

Admin → Server

```json
{
  "type": "KICK_FROM_WORLD",
  "payload": {
    "target_user_id": "uuid",
    "world_id": "uuid"
  }
}
```

---

## 16. Versioning

The protocol version is included in every message envelope.

* Protocol version 1: Initial version
* Breaking changes increment the major version
* Non-breaking additions increment the minor version

Clients and servers must negotiate compatibility during connection.

---

## 17. Principles

* Signaling is not media
* Minimal state
* Fail fast
* Client is responsible for media
* Server is authoritative for validation
* All state changes flow through Signaling Server

---

## 18. Message Ordering

* Messages are delivered in order per WebSocket connection
* Presence updates may be batched by the server
* Critical messages (errors, corrections) are sent immediately

---

## Final Note

This protocol is designed to be:

* Simple to implement
* Easy to debug
* Extensible without breaking changes
* Compatible with horizontal scaling

Any runtime behavior not derivable from this protocol is considered a bug.

