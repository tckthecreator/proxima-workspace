# Signaling Protocol – WebRTC Coordination

Este documento define o **protocolo de signaling** usado para coordenar conexões WebRTC, estados de sala e transições entre P2P, TURN e SFU.

O servidor de signaling **não transmite mídia**. Ele apenas coordena peers e decisões de topologia.

Este protocolo é **normativo**: client e servidor devem implementá-lo conforme descrito.

---

## 1. Objetivos do Signaling

* Descoberta de peers
* Troca de SDP e ICE candidates
* Coordenação de ZoneType
* Promoção automática para SFU
* Sincronização de estado mínimo

---

## 2. Transporte

* WebSocket (JSON)
* Uma conexão por client
* Estado efêmero (sem persistência)

---

## 3. Identidades

### 3.1 User

```json
{
  "user_id": "uuid",
  "display_name": "string",
  "team_id": "uuid"
}
```

### 3.2 Room

```json
{
  "room_id": "uuid",
  "zone_type": "OPEN | PERSONAL | TEAM_AREA | MEETING | AFK",
  "overrides": {}
}
```

---

## 4. Estados de Conexão

### 4.1 ClientConnectionState

* `CONNECTING`
* `CONNECTED`
* `DISCONNECTED`

### 4.2 RoomMediaState

* `P2P_DIRECT`
* `P2P_TURN`
* `SFU`

---

## 5. Mensagens Base

Todas as mensagens seguem o envelope:

```json
{
  "type": "string",
  "payload": {}
}
```

---

## 6. Fluxo Inicial

### 6.1 Join Room

Client → Server

```json
{
  "type": "JOIN_ROOM",
  "payload": {
    "room_id": "uuid"
  }
}
```

Server → Client

```json
{
  "type": "ROOM_STATE",
  "payload": {
    "room": { /* Room */ },
    "media_state": "P2P_DIRECT",
    "peers": []
  }
}
```

---

## 7. Peer Discovery

Server → Client

```json
{
  "type": "PEER_JOINED",
  "payload": {
    "user": { /* User */ }
  }
}
```

Server → Client

```json
{
  "type": "PEER_LEFT",
  "payload": {
    "user_id": "uuid"
  }
}
```

---

## 8. WebRTC Negotiation (P2P)

### 8.1 SDP Offer

Client → Server

```json
{
  "type": "SDP_OFFER",
  "payload": {
    "to": "user_id",
    "sdp": "base64"
  }
}
```

Server → Client

```json
{
  "type": "SDP_OFFER",
  "payload": {
    "from": "user_id",
    "sdp": "base64"
  }
}
```

---

### 8.2 SDP Answer

(similar)

---

### 8.3 ICE Candidate

```json
{
  "type": "ICE_CANDIDATE",
  "payload": {
    "to": "user_id",
    "candidate": {}
  }
}
```

---

## 9. TURN Fallback

Client → Server

```json
{
  "type": "ICE_FAILED",
  "payload": {
    "peer_id": "user_id"
  }
}
```

Server decide fallback para TURN individual.

---

## 10. Promoção para SFU

Server → All Clients

```json
{
  "type": "PROMOTE_TO_SFU",
  "payload": {
    "sfu_url": "wss://sfu.example.com"
  }
}
```

Clients conectam ao SFU em paralelo e migram mídia.

---

## 11. SFU Connection

Client → Server

```json
{
  "type": "SFU_CONNECTED"
}
```

---

## 12. Zone Transitions

Client → Server

```json
{
  "type": "ZONE_CHANGE",
  "payload": {
    "zone_type": "MEETING"
  }
}
```

Server valida e propaga.

---

## 13. Estados do Usuário

Client → Server

```json
{
  "type": "USER_STATUS",
  "payload": {
    "status": "AFK | ONLINE | DND"
  }
}
```

---

## 14. Erros

```json
{
  "type": "ERROR",
  "payload": {
    "code": "string",
    "message": "string"
  }
}
```

---

## 15. Versionamento

O protocolo deve incluir:

```json
{
  "protocol_version": 1
}
```

Mudanças incompatíveis incrementam a versão.

---

## 16. Princípios

* Signaling não é mídia
* Estado mínimo
* Fail fast
* Cliente é responsável por mídia

---

Este documento deve evoluir junto com `ARCHITECTURE_GUIDE.md` e `INTERACTION_DESIGN.md`.
