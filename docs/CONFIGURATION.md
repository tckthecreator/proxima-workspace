# CONFIGURATION.md

This document defines the **configuration schemas** for all system components, including Signaling Server, SFU, TURN, Client, and deployment configurations.

---

## Related Documents

This document should be read **after**:

* README.md
* ARCHITECTURE.md
* SECURITY_MODEL.md

---

## Principles

* Configuration via environment variables and files
* Sensitive values via environment variables only
* Default values for all settings
* Validation on startup
* Docker-friendly configuration

---

## Signaling Server Configuration

### Environment Variables

```bash
# Server
SIGNALING_PORT=8080
SIGNALING_HOST=0.0.0.0
SIGNALING_LOG_LEVEL=info

# Database
DATABASE_URL=postgresql://user:pass@localhost/proxima
DATABASE_POOL_SIZE=10
DATABASE_MAX_CONNECTIONS=100

# Redis (optional, for presence caching)
REDIS_URL=redis://localhost:6379
REDIS_PASSWORD=

# Authentication
JWT_SECRET=your-secret-key
JWT_ISSUER=proxima
JWT_ACCESS_TTL=15m
JWT_REFRESH_TTL=7d

# Rate Limiting
RATE_LIMIT_ENABLED=true
RATE_LIMIT_RPS=100
RATE_LIMIT_BURST=200

# CORS
CORS_ALLOWED_ORIGINS=https://app.proxima.example.com
CORS_ALLOW_CREDENTIALS=true

# Metrics
METRICS_ENABLED=true
METRICS_PORT=9090
```

### Configuration File (config.yaml)

```yaml
server:
  port: 8080
  host: "0.0.0.0"
  log_level: "info"
  
database:
  url: "${DATABASE_URL}"
  pool_size: 10
  max_connections: 100
  ssl_mode: "prefer"
  
redis:
  url: "${REDIS_URL}"
  password: "${REDIS_PASSWORD}"
  enabled: false
  
auth:
  jwt_secret: "${JWT_SECRET}"
  jwt_issuer: "proxima"
  access_token_ttl: "15m"
  refresh_token_ttl: "7d"
  
rate_limiting:
  enabled: true
  requests_per_second: 100
  burst_size: 200
  
cors:
  allowed_origins:
    - "https://app.proxima.example.com"
  allow_credentials: true
  
metrics:
  enabled: true
  port: 9090
  
presence:
  update_frequency_max: 15  # Hz
  broadcast_frequency: 10   # Hz
  zone_snapshot_on_transition: true
  
media:
  p2p_max_participants: 8
  sfu_threshold: 12
  sfu_cpu_threshold: 70  # %
  sfu_packet_loss_threshold: 5  # %
  sfu_turn_usage_threshold: 30  # %
  
limits:
  max_users_per_world: 150
  max_users_per_zone:
    OpenArea: 50
    TeamArea: 25
    MeetingRoom: 16
    PrivateRoom: 8
  max_objects_per_map: 2000
```

---

## SFU Configuration

### Environment Variables

```bash
# Server
SFU_PORT=8081
SFU_HOST=0.0.0.0
SFU_LOG_LEVEL=info

# Signaling Server
SIGNALING_URL=ws://signaling:8080
SIGNALING_TOKEN=sfu-token

# WebRTC
ICE_SERVERS=stun:stun.l.google.com:19302
TURN_URL=turn:turn.example.com:3478
TURN_USERNAME=turn-user
TURN_PASSWORD=turn-pass

# Limits
SFU_MAX_STREAMS=100
SFU_MAX_BANDWIDTH_PER_STREAM=2000000  # 2 Mbps

# Metrics
METRICS_ENABLED=true
METRICS_PORT=9091
```

### Configuration File (sfu-config.yaml)

```yaml
server:
  port: 8081
  host: "0.0.0.0"
  log_level: "info"
  
signaling:
  url: "${SIGNALING_URL}"
  token: "${SIGNALING_TOKEN}"
  
webrtc:
  ice_servers:
    - urls: "stun:stun.l.google.com:19302"
    - urls: "${TURN_URL}"
      username: "${TURN_USERNAME}"
      credential: "${TURN_PASSWORD}"
  
limits:
  max_streams: 100
  max_bandwidth_per_stream: 2000000  # 2 Mbps
  max_bitrate_audio: 64000   # 64 kbps
  max_bitrate_video: 2000000 # 2 Mbps
  
metrics:
  enabled: true
  port: 9091
```

---

## TURN Server Configuration

TURN server implemented in Go using pion/turn for single-language stack consistency.

### Environment Variables

```bash
# Server
TURN_PORT=3478
TURN_HOST=0.0.0.0
TURN_LOG_LEVEL=info

# Network
EXTERNAL_IP=your-public-ip
REALM=proxima.example.com

# Authentication
TURN_SECRET=your-secret-key

# Limits
TURN_MAX_BPS=10000000  # 10 Mbps
TURN_MAX_ALLOCATE_LIFETIME=3600
TURN_MAX_ALLOCATE_TIMEOUT=60
```

### Configuration File (turn-config.yaml)

```yaml
server:
  port: 3478
  host: "0.0.0.0"
  log_level: "info"
  
network:
  external_ip: "${EXTERNAL_IP}"
  realm: "proxima.example.com"
  
auth:
  secret: "${TURN_SECRET}"
  # Or use database-backed auth
  
limits:
  max_bps: 10000000  # 10 Mbps
  max_allocate_lifetime: 3600
  max_allocate_timeout: 60
  
security:
  no_cli: true
  no_tcp_relay: false  # Enable if needed
  no_multicast_peers: true
```

### Go Implementation Example

```go
// Example TURN server setup using pion/turn
turnConfig := turn.ServerConfig{
    Realm: "proxima.example.com",
    AuthHandler: func(username, realm string, srcAddr net.Addr) ([]byte, bool) {
        // Validate credentials (can integrate with signaling server auth)
        return validateCredentials(username, realm)
    },
    ListenerConfig: &turn.ListenerConfig{
        ListeningPort: 3478,
        RelayAddressGenerator: &turn.RelayAddressGeneratorStatic{
            RelayAddress: net.ParseIP(os.Getenv("EXTERNAL_IP")),
            Address:      "0.0.0.0",
        },
    },
}
```

**Benefits of Go TURN:**
* Single language stack (consistent with signaling server)
* Easier integration and shared code
* Unified observability (same metrics/logging framework)
* Simpler deployment (single Go binary)
* Better maintainability (one codebase, one toolchain)

---

## Client Configuration

### Runtime Configuration (client-config.json)

```json
{
  "signaling": {
    "url": "wss://signaling.proxima.example.com",
    "reconnect_attempts": 10,
    "reconnect_delay_ms": 1000,
    "reconnect_max_delay_ms": 30000
  },
  "media": {
    "audio": {
      "enabled": true,
      "constraints": {
        "echoCancellation": true,
        "noiseSuppression": true,
        "autoGainControl": true
      }
    },
    "video": {
      "enabled": false,
      "constraints": {
        "width": { "ideal": 1280 },
        "height": { "ideal": 720 },
        "frameRate": { "ideal": 30 }
      }
    }
  },
  "presence": {
    "update_frequency": 15,
    "position_precision": 1
  },
  "rendering": {
    "tile_size": 32,
    "fps_target": 60,
    "enable_shadows": true,
    "enable_animations": true
  },
  "limits": {
    "max_concurrent_media_tracks": 10,
    "max_audio_tracks": 8,
    "max_video_tracks": 2
  }
}
```

### Build-time Configuration

```typescript
// config.ts
export const config = {
  apiUrl: process.env.REACT_APP_API_URL || 'https://api.proxima.example.com',
  wsUrl: process.env.REACT_APP_WS_URL || 'wss://signaling.proxima.example.com',
  cdnUrl: process.env.REACT_APP_CDN_URL || 'https://cdn.proxima.example.com',
  version: process.env.REACT_APP_VERSION || '1.0.0',
  environment: process.env.NODE_ENV || 'production'
};
```

---

## Docker Compose Configuration

### docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: proxima
      POSTGRES_USER: proxima
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U proxima"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  signaling:
    build: ./signaling
    environment:
      DATABASE_URL: postgresql://proxima:${POSTGRES_PASSWORD}@postgres:5432/proxima
      REDIS_URL: redis://redis:6379
      JWT_SECRET: ${JWT_SECRET}
    ports:
      - "8080:8080"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./config/signaling.yaml:/app/config.yaml:ro

  sfu:
    build: ./sfu
    environment:
      SIGNALING_URL: ws://signaling:8080
      SIGNALING_TOKEN: ${SFU_TOKEN}
      TURN_URL: turn:turn:3478
      TURN_USERNAME: ${TURN_USERNAME}
      TURN_PASSWORD: ${TURN_PASSWORD}
    ports:
      - "8081:8081"
    depends_on:
      - signaling
    volumes:
      - ./config/sfu-config.yaml:/app/config.yaml:ro

  turn:
    build: ./turn
    environment:
      EXTERNAL_IP: ${EXTERNAL_IP}
      TURN_SECRET: ${TURN_SECRET}
      REALM: proxima.example.com
    ports:
      - "3478:3478/udp"
      - "3478:3478/tcp"
      - "5349:5349/udp"
      - "5349:5349/tcp"
    volumes:
      - ./config/turn-config.yaml:/app/config.yaml:ro
    depends_on:
      - signaling

  asset-server:
    image: nginx:alpine
    ports:
      - "8082:80"
    volumes:
      - ./assets:/usr/share/nginx/html:ro
      - ./config/nginx.conf:/etc/nginx/nginx.conf:ro

volumes:
  postgres_data:
  redis_data:
```

### Environment File (.env)

```bash
# Database
POSTGRES_PASSWORD=change-me

# JWT
JWT_SECRET=change-me-to-random-secret

# SFU
SFU_TOKEN=change-me-to-random-token

# TURN
EXTERNAL_IP=your-public-ip
TURN_USERNAME=turn-user
TURN_PASSWORD=change-me

# Optional
REDIS_PASSWORD=
```

---

## Configuration Validation

### Validation Rules

* All required fields must be present
* Ports must be valid (1-65535)
* URLs must be valid format
* Timeouts must be positive integers
* Thresholds must be valid percentages (0-100)

### Startup Validation

* Configuration validated on startup
* Invalid configuration causes startup failure
* Clear error messages for invalid values
* Default values used when optional fields missing

---

## Configuration Management

### Environment-Specific Configs

* `config.dev.yaml` - Development
* `config.staging.yaml` - Staging
* `config.prod.yaml` - Production

### Secrets Management

* Sensitive values via environment variables
* Never commit secrets to version control
* Use secret management tools (Vault, AWS Secrets Manager, etc.)
* Rotate secrets regularly

---

## Explicit Rules

> All configuration must have defaults.

> Sensitive values must come from environment variables.

> Configuration must be validated on startup.

> Configuration changes require restart (no hot-reload).

> Docker configurations must be self-contained.

---

## Final Note

This configuration system ensures:

* Easy deployment
* Secure defaults
* Clear documentation
* Validation at startup
* Docker-friendly

All components should follow these configuration patterns for consistency and maintainability.

