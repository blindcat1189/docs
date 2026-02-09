# Multi-Location Partner Integration - Quick Reference

> **See Also**: [Full Design Doc](./multi-location-partner-integration-design.md) | [ADR-001](./decisions/adr-001-multi-location-partner-integration.md)

---

## TL;DR

Add multi-location support to Nexus for chain restaurants accessing external partners (PIE first). Single-store flow unchanged
chain users get location selection.

---

## What's New

### New Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/v1/advertising/locations` | List user's accessible locations |
| POST | `/api/v1/advertising/multi_location/sso_token` | Multi-location SSO token |

### Unchanged Endpoint

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/api/v1/advertising/sso_token` | Single-location SSO (backward compat) |

---

## User Flows

### Single-Store User
```
Request → /sso_token → Token (1 location) → PIE
```

### Chain User (Multiple Locations)
```
Request → /locations → Select stores → /multi_location/sso_token → Token (N locations) → PIE
```

---

## JWT Token Changes

**Single-location (unchanged):**
```json
{
  "partner_id": "",
  "partner_location_id": "12345"
}
```

**Multi-location (new):**
```json
{
  "partner_id": "99999",
  "partner_location_id": "12345",
  "partner_location_ids": ["12345", "12346", "12347"],
  "access_scope": "MULTI"
}
```

---

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Chain Tenant** | Brand with multiple locations |
| **User Perspective** | HQ (sees multiple stores) vs STORE (sees one) |
| **Jurisdiction Scope** | Which locations a user can access |
| **Access Scope** | SINGLE, MULTI, or ALL |

---

## New Files

### DTOs (nexus-client)
- `LocationInfo.java`
- `ChainInfo.java`
- `UserLocationAccessResponse.java`
- `MultiLocationSsoRequest.java`
- `MultiLocationSsoTokenResponse.java`

### Constants (nexus-common)
- `AccessScope.java` enum
- `UserPerspective.java` enum

### Services (nexus-integration)
- `ChowbusChainService.java`

### Strategy (nexus-service)
- `PieTokenStrategy.java` interface
- `SingleTokenStrategy.java`
- `MultiTokenStrategy.java`

---

## Implementation Phases

| Phase | Can Start Now? | Description |
|-------|---------------|-------------|
| 1 | Yes | DTOs and constants |
| 2 | **No** (needs API specs) | Integration layer |
| 3 | Partial | Service interfaces |
| 4 | Partial | Controller structure |
| 5 | Yes | Configuration |

---

## Blockers

| Item | Owner | Status |
|------|-------|--------|
| Internal APIs for chain data | Backend Team | **Pending** |
| PIE token format confirmation | PIE Team | Pending (not blocking) |

---

## Error Codes

| Code | Name | HTTP Status |
|------|------|-------------|
| 310001 | LOCATION_ACCESS_DENIED | 403 |
| 310002 | CHAIN_NOT_FOUND | 404 |
| 310003 | NO_ACCESSIBLE_LOCATIONS | 400 |

---

## Testing Commands

```bash
# Get locations
curl -X GET http://localhost:9001/api/v1/advertising/locations \
  -H "Authorization: Bearer <jwt>"

# Multi-location token
curl -X POST http://localhost:9001/api/v1/advertising/multi_location/sso_token \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -d '{"chainId": 99999, "locationIds": [12345, 12346]}'
```

---

## Config

```yaml
pie:
  multi-location:
    enabled: true
    max-locations-per-request: 50
    token-strategy: single  # or multi
```

---

*Last Updated: 2026-01-19*
