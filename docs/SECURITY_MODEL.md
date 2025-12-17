# SECURITY_MODEL.md

This document defines the **security model** for authentication, authorization, token management, and workspace isolation.

---

## Related Documents

This document should be read **after**:

* README.md
* ARCHITECTURE.md
* PRODUCT_MODEL.md
* SIGNALING_PROTOCOL.md

---

## Principles

* Authentication required for all operations
* Authorization based on roles and permissions
* Workspace isolation is strict
* Tokens are short-lived and refreshable
* All sensitive operations are logged

---

## Authentication

### Authentication Flow

1. **User Login**
   * User provides credentials (email/password, OAuth, SSO)
   * Authentication server validates credentials
   * Authentication server issues JWT access token and refresh token

2. **Client Connection**
   * Client connects to Signaling Server with access token
   * Signaling Server validates token signature and expiration
   * Signaling Server establishes authenticated session

3. **Token Refresh**
   * Client uses refresh token to obtain new access token
   * Refresh token validated by authentication server
   * New access token issued

### Token Structure

**Access Token (JWT)**

```json
{
  "user_id": "uuid",
  "workspace_id": "uuid",
  "roles": ["Member", "Editor"],
  "exp": 1234567890,
  "iat": 1234567800
}
```

**Refresh Token**

* Long-lived token (7-30 days)
* Stored securely (httpOnly cookie recommended)
* Used only for token refresh
* Revocable

### Token Validation

* Signature verification (RS256 or HS256)
* Expiration check
* Issuer validation
* Audience validation
* Workspace membership validation

---

## Authorization

### Roles

**Admin**
* Full workspace control
* User management
* World management
* Moderation actions
* Billing access

**Editor**
* World editing (Build Mode)
* Asset management
* Mute users
* Cannot delete worlds or users

**Member**
* Join worlds
* Interact with objects
* Use collaboration tools
* Cannot edit worlds

**Guest**
* Limited access (if enabled)
* View-only in some cases
* Time-limited sessions

### Permissions Matrix

| Action | Admin | Editor | Member | Guest |
|--------|-------|--------|--------|-------|
| Join World | ✅ | ✅ | ✅ | ⚠️ |
| Edit World | ✅ | ✅ | ❌ | ❌ |
| Mute User | ✅ | ✅ | ❌ | ❌ |
| Remove from Zone | ✅ | ❌ | ❌ | ❌ |
| Kick from World | ✅ | ❌ | ❌ | ❌ |
| Lock Door | ✅ | ❌ | ❌ | ❌ |
| Upload Assets | ✅ | ✅ | ❌ | ❌ |
| Delete World | ✅ | ❌ | ❌ | ❌ |
| Manage Users | ✅ | ❌ | ❌ | ❌ |

### Permission Checks

* **Client-side**: UI reflects permissions (UX only)
* **Server-side**: All operations validated (authoritative)
* **Fail-fast**: Deny immediately if unauthorized

---

## Workspace Isolation

### Isolation Rules

* Users can only access their workspace's worlds
* Worlds are scoped to workspace
* Assets are scoped to workspace
* Presence is scoped to workspace
* No cross-workspace operations

### Enforcement

* Token includes `workspace_id`
* All requests validated against token workspace
* Database queries filtered by workspace
* API endpoints validate workspace membership

### Cross-Workspace Scenarios

* **Not supported**: Users cannot be in multiple workspaces simultaneously
* **Avatar scope**: Avatars are workspace-specific (per PRODUCT_MODEL.md)
* **Asset sharing**: Assets cannot be shared across workspaces

---

## Token Management

### Access Token Lifetime

* Default: 15 minutes
* Maximum: 1 hour
* Short-lived for security

### Refresh Token Lifetime

* Default: 7 days
* Maximum: 30 days
* Revocable on logout or security event

### Token Revocation

Tokens are revoked when:
* User logs out
* Password changed
* Security event detected
* Admin revokes access
* Workspace membership revoked

### Token Storage

**Client-side:**
* Access token: Memory only (not localStorage)
* Refresh token: Secure httpOnly cookie (preferred) or secure storage

**Server-side:**
* Token blacklist (for revocation)
* Token validation cache

---

## API Security

### Transport Security

* **HTTPS/WSS required**: All connections encrypted
* **TLS 1.2+**: Minimum TLS version
* **Certificate pinning**: Optional for mobile clients

### Request Validation

* **Rate limiting**: Per user, per endpoint
* **Input validation**: All inputs validated
* **CSRF protection**: For state-changing operations
* **Origin validation**: CORS properly configured

### Rate Limiting

* **Presence updates**: 15 Hz maximum
* **Movement intents**: 30 Hz maximum
* **Object interactions**: 10 requests/second
* **Authentication**: 5 attempts/minute

---

## Data Security

### Data at Rest

* **Encryption**: Sensitive data encrypted
* **Hashing**: Passwords hashed (bcrypt/argon2)
* **Backup encryption**: Backups encrypted

### Data in Transit

* **TLS**: All connections encrypted
* **WebRTC**: DTLS for media
* **Signaling**: WSS for signaling

### Data Privacy

* **User data**: Only accessible by authorized users
* **Presence data**: Only visible to same zone
* **World data**: Only accessible by workspace members
* **Logs**: No sensitive data in logs

---

## Moderation & Abuse Prevention

### Moderation Actions

All moderation actions are logged:

* User muted
* User removed from zone
* User kicked from world
* Door locked
* World edited

### Abuse Prevention

* **Rate limiting**: Prevents spam
* **Content filtering**: Asset moderation
* **Behavioral analysis**: Anomaly detection
* **Reporting**: User reporting system

### Audit Logging

All administrative actions logged:
* Who performed action
* What action was performed
* When action occurred
* Target of action
* Reason (if provided)

---

## Security Events

### Security Event Types

* **Authentication failure**: Failed login attempts
* **Authorization failure**: Unauthorized access attempts
* **Token theft**: Token used from unexpected location
* **Abuse detection**: Suspicious behavior patterns

### Response to Security Events

* **Immediate**: Revoke affected tokens
* **Short-term**: Rate limit user
* **Long-term**: Review and update security policies

---

## Compliance

### Data Retention

* **User data**: Retained while account active
* **Logs**: Retained for 90 days
* **Audit logs**: Retained for 1 year
* **Backups**: Retained per backup policy

### User Rights

* **Data access**: Users can request their data
* **Data deletion**: Users can request deletion
* **Account deletion**: Users can delete accounts
* **Export**: Users can export their data

---

## Explicit Rules

> All authentication and authorization decisions are made server-side.

> Client-side permission checks are for UX only, not security.

> Workspace isolation is strict and enforced at all layers.

> Tokens are short-lived and refreshable.

> All security events are logged and monitored.

---

## Final Note

Security is a foundational concern. All components must:

* Validate authentication
* Enforce authorization
* Maintain workspace isolation
* Log security events
* Handle failures securely

This model ensures that the system is secure by design, not by accident.

