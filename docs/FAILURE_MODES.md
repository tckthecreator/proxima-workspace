# FAILURE_MODES.md

This document describes the **expected failure modes** of the system and how the Client and infrastructure should react to each. The goal is to ensure **graceful degradation**, predictability, and automatic recovery whenever possible.

---

## Related Documents

This document should be read **after**:

* README.md
* ARCHITECTURE.md
* CLIENT_STATE_MACHINE.md
* SIGNALING_PROTOCOL.md

---

## Principles

* Failure is an expected state
* Prefer degradation over total interruption
* Automatic recovery whenever possible
* Safe states above media continuity
* Fail fast, recover gracefully

---

## Failure Categories

1. Connectivity Failures
2. Media Failures (WebRTC)
3. SFU Failures
4. TURN Failures
5. Signaling Failures
6. Client Failures
7. Server Failures

---

## 1. Connectivity Failures

### Symptoms

* Temporary network loss
* High latency (> 500ms)
* Elevated packet loss (> 5%)
* Intermittent connectivity

### Expected Behavior

* Suspend media transmission
* Maintain local state
* Display reconnection status
* Continue presence updates when possible
* Preserve object locks (with timeout)

### Recovery

* Automatic reconnection attempt
* Exponential backoff (1s, 2s, 4s, 8s, max 30s)
* Re-evaluate policy (P2P → TURN → SFU)
* Resume media when connection restored

### Timeouts

* Presence update timeout: 5 seconds
* Object lock timeout: 10 seconds
* Full disconnect timeout: 30 seconds

---

## 2. Media Failures (WebRTC)

### Symptoms

* ICE connection failed
* Track ended unexpectedly
* Audio/video frozen
* High jitter or latency

### Expected Behavior

* Close only the affected track
* Keep signaling connection active
* Notify user non-intrusively
* Maintain presence state
* Continue other media tracks if possible

### Recovery

* Automatic renegotiation
* Connectivity fallback (P2P → TURN → SFU)
* Track restart with new connection
* Maximum 3 retry attempts

### Error Codes

* `ICE_FAILED` - NAT traversal failed
* `TRACK_ENDED` - Media track ended unexpectedly
* `CONNECTION_TIMEOUT` - Connection timeout

---

## 3. SFU Failures

### Symptoms

* SFU disconnection
* Excessive delay (> 1s)
* Missing streams
* High packet loss from SFU

### Expected Behavior

* Disable SFU for the zone
* Maintain presence
* Fallback to P2P if allowed by ZoneType
* Notify participants (non-intrusive)

### Recovery

* Fallback to P2P (when permitted)
* Automatic re-entry to SFU when available
* Re-evaluate zone metrics
* Seamless transition (no media drop)

### ZoneType-Specific Behavior

* **MeetingRoom**: Attempt immediate SFU reconnection
* **TeamArea**: Fallback to P2P, promote to SFU when available
* **OpenArea**: Never uses SFU, no fallback needed
* **PrivateRoom**: Fallback to P2P

---

## 4. TURN Failures

### Symptoms

* TURN server unreachable
* High relay costs
* TURN connection timeout
* Authentication failure

### Expected Behavior

* Avoid excessive TURN usage
* Notify backend
* Attempt direct P2P connection
* Use SFU if available

### Recovery

* Retry direct P2P connection
* Use SFU if available
* Request new TURN credentials
* Maximum 2 retry attempts

---

## 5. Signaling Failures

### Symptoms

* WebSocket closed unexpectedly
* Message timeout
* Authentication failure
* Server error responses

### Expected Behavior

* Enter Connecting state
* Suspend all media
* Preserve local state (position, preferences)
* Display reconnection UI

### Recovery

* Exponential backoff reconnection (1s, 2s, 4s, 8s, 16s, 30s)
* Full state resync on reconnect
* Re-authenticate if token expired
* Maximum 10 retry attempts

### Error Codes

* `AUTH_FAILED` - Token invalid or expired
* `SERVER_ERROR` - Internal server error
* `RATE_LIMITED` - Too many requests
* `CONNECTION_CLOSED` - Server closed connection

---

## 6. Client Failures

### Symptoms

* Excessive CPU usage (> 90%)
* Browser tab frozen
* Memory exhaustion
* Client crash

### Expected Behavior

* Aggressively close media tracks
* Save minimal state (preferences)
* Enter Disconnected state
* Log error details

### Recovery

* Client restart required
* Safe re-entry to Idle state
* Restore user preferences
* Reconnect to last world

### Prevention

* Monitor CPU usage
* Limit concurrent media tracks
* Implement resource limits
* Graceful degradation under load

---

## 7. Server Failures

### Symptoms

* Signaling server unreachable
* High response latency
* 5xx HTTP errors
* Service degradation

### Expected Behavior

* Client enters Connecting state
* Suspend all operations
* Display service unavailable message
* Preserve local state

### Recovery

* Automatic reconnection with backoff
* State resync on reconnect
* Re-authenticate if needed
* Maximum 10 retry attempts

---

## Failure Handling by ZoneType

### OpenArea

* Prefer disabling audio
* Maintain presence
* No SFU fallback needed

### TeamArea

* Prioritize SFU recovery
* Fallback to P2P acceptable
* Notify team members

### MeetingRoom

* Notify all participants
* Coordinate re-entry
* Prioritize SFU recovery
* Consider meeting pause

### PrivateRoom

* Maintain local control
* Preserve privacy
* Notify authorized participants only

---

## Retry Strategies

### Exponential Backoff

* Initial delay: 1 second
* Maximum delay: 30 seconds
* Multiplier: 2x per attempt
* Maximum attempts: 10

### Circuit Breaker

* After 5 consecutive failures: Open circuit
* Circuit open duration: 60 seconds
* Half-open state: Test connection
* Close circuit on success

---

## Observability

### Required Events

* `failure_detected` - Failure identified
* `recovery_attempted` - Recovery initiated
* `recovery_success` - Recovery succeeded
* `recovery_failed` - Recovery failed
* `state_changed` - State transition due to failure

### Metrics

* Failure rate by type
* Recovery time
* Success rate
* Mean time to recovery (MTTR)

---

## Degradation Policies

### Audio Degradation

1. Reduce bitrate
2. Disable video
3. Reduce update frequency
4. Disable audio (last resort)

### Video Degradation

1. Reduce resolution
2. Reduce framerate
3. Disable video (keep audio)
4. Full degradation

### Presence Degradation

1. Reduce update frequency (15 Hz → 5 Hz)
2. Reduce precision (round positions)
3. Disable presence (last resort)

---

## Explicit Rules

> Failures are part of the normal system flow.

> Product robustness is measured not by the absence of failures, but by the **quality of recovery**.

> All failures must return to safe states (Idle or Disconnected).

> Media continuity is secondary to system stability.

---

## Final Note

This document defines how the system handles expected failures. Unhandled failures should:

1. Log detailed error information
2. Enter safe state (Disconnected)
3. Notify user appropriately
4. Allow manual recovery

The system is designed to degrade gracefully rather than fail catastrophically.

