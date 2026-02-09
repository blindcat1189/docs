# Multi-Location Account Management - Partner Integration Technical Design

> **Status**: Draft
> **Author**: Engineering Team
> **Created**: 2026-01-19
> **Last Updated**: 2026-01-19

---

## Table of Contents

1. [Problem Definition](#1-problem-definition)
2. [Tenant Model](#2-tenant-model)
3. [Role & Permission Model](#3-role--permission-model)
4. [User-Location Access Model](#4-user-location-access-model)
5. [Partner Integration Interface](#5-partner-integration-interface)
6. [Solution Approaches](#6-solution-approaches)
7. [Architecture Diagrams](#7-architecture-diagrams)
8. [User Flow Diagrams](#8-user-flow-diagrams)
9. [PIE Implementation](#9-pie-implementation-first-consumer)
10. [Implementation Plan](#10-implementation-plan)
11. [Dependencies & Open Questions](#11-dependencies--open-questions)

---

## 1. Problem Definition

### 1.1 Current State

Nexus currently has **no multi-location infrastructure** for partner integrations:

| Aspect | Current Implementation |
|--------|----------------------|
| `partner_location_id` | Single restaurant ID only |
| `partner_id` | Empty/unused field |
| Chain/Tenant hierarchy | Not supported |
| User-location access control | Not implemented |

### 1.2 Business Need

Chowbus needs to support:

1. **Chain/Franchise Tenant Types**
   - Single-store tenants (independent restaurants)
   - Chain tenants (HQ + multiple store locations)

2. **Per-User Location Access Control**
   - Same role can have different location scopes per user
   - Not just per-role, but per-user granularity

3. **Consistent Partner Mapping**
   - Generic interface for any external partner system
   - First implementation: PIE (advertising platform)

### 1.3 Target Users

```
┌─────────────────────────────────────────────────────────┐
│                    CHAIN TENANT                          │
├──────────────────────┬──────────────────────────────────┤
│       HQ Users       │         Store Users              │
├──────────────────────┼──────────────────────────────────┤
│ • Boss (Owner)       │ • Store Manager                  │
│ • Operations         │ • Staff                          │
│ • Marketing          │                                  │
│ • IT Administrator   │                                  │
│ • Supply Chain       │                                  │
└──────────────────────┴──────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                 SINGLE-STORE TENANT                      │
├─────────────────────────────────────────────────────────┤
│ • Boss/Owner                                            │
│ • Manager                                               │
│ • Staff                                                 │
└─────────────────────────────────────────────────────────┘
```

---

## 2. Tenant Model

### 2.1 Single-Store Tenant (独立门店)

```
┌─────────────────────────────────┐
│       Single Restaurant         │
│                                 │
│  ┌───────────────────────────┐  │
│  │    One Perspective        │  │
│  │    (Store-level only)     │  │
│  └───────────────────────────┘  │
│                                 │
│  Users: Boss, Manager, Staff    │
└─────────────────────────────────┘
```

- One restaurant = one perspective
- All users see the same store data
- Simple permission model

### 2.2 Chain Tenant (连锁品牌)

```
┌──────────────────────────────────────────────────────────┐
│                    CHAIN BRAND                            │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │              HQ PERSPECTIVE (总部视角)              │  │
│  │                                                    │  │
│  │  • See all stores or subset of stores             │  │
│  │  • Aggregate reporting                            │  │
│  │  • Cross-location operations                      │  │
│  └────────────────────────────────────────────────────┘  │
│                          │                               │
│          ┌───────────────┼───────────────┐               │
│          ▼               ▼               ▼               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐        │
│  │   Store A   │ │   Store B   │ │   Store C   │        │
│  │  (门店视角)  │ │  (门店视角)  │ │  (门店视角)  │        │
│  └─────────────┘ └─────────────┘ └─────────────┘        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

- One brand = multiple stores
- Two perspectives: HQ (总部) and Store (门店)
- Role determines what you can do
location scope determines where

### 2.3 Tenant Type Identification

| Tenant Type | Chain ID | Location Count |
|-------------|----------|----------------|
| Single-Store | null | 1 |
| Chain | non-null | 2+ |

---

## 3. Role & Permission Model

### 3.1 HQ-Level Roles (总部角色)

| Role | Chinese | Description | Typical Permissions |
|------|---------|-------------|---------------------|
| Boss | 老板 | Chain owner | Full access to all locations |
| Operations | 运营 | Operations manager | Operations across locations |
| Marketing | 市场 | Marketing manager | Marketing/advertising access |
| IT Admin | IT管理员 | System administrator | Technical/admin functions |
| Supply Chain | 供应链 | Supply chain manager | Inventory, procurement |
| Finance | 财务 | Financial controller | Financial data, reporting |

### 3.2 Store-Level Roles (门店角色)

| Role | Chinese | Description | Typical Permissions |
|------|---------|-------------|---------------------|
| Store Manager | 店长 | Store manager | Full store access |
| Staff | 员工 | General staff | Limited store access |

### 3.3 Permission Scopes

```
┌─────────────────────────────────────────────────────────┐
│                   PERMISSION MATRIX                      │
├─────────────────┬───────────────────────────────────────┤
│   Permission    │              Scope                     │
├─────────────────┼───────────────────────────────────────┤
│ advertising:read│ View advertising data                 │
│ advertising:write│ Create/modify campaigns              │
│ reports:view    │ View reports                          │
│ settings:manage │ Manage settings                       │
└─────────────────┴───────────────────────────────────────┘
```

---

## 4. User-Location Access Model

### 4.1 Core Concept: 管辖范围 (Jurisdiction Scope)

**Key insight**: Same role, different location scope per user.

```
┌─────────────────────────────────────────────────────────┐
│                 CHAIN: "Happy Dumpling"                  │
│                                                          │
│  Locations: NYC, LA, Chicago, Houston, Seattle          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  User A (Marketing Manager)                             │
│  ├─ Role: Marketing                                     │
│  └─ Accessible Locations: [NYC, LA, Chicago]            │
│                                                          │
│  User B (Marketing Manager)                             │
│  ├─ Role: Marketing                                     │
│  └─ Accessible Locations: [Houston, Seattle]            │
│                                                          │
│  User C (Boss)                                          │
│  ├─ Role: Boss                                          │
│  └─ Accessible Locations: [ALL]                         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Access Control Data Model

```
┌─────────────────────────────────────────────────────────┐
│                USER_LOCATION_ACCESS                      │
├─────────────┬─────────────┬─────────────┬───────────────┤
│   user_id   │  chain_id   │ location_id │ access_type   │
├─────────────┼─────────────┼─────────────┼───────────────┤
│     101     │    9999     │    NULL     │ ALL           │
│     102     │    9999     │    12345    │ SINGLE        │
│     102     │    9999     │    12346    │ SINGLE        │
│     103     │    9999     │    12347    │ SINGLE        │
└─────────────┴─────────────┴─────────────┴───────────────┘

access_type:
  - ALL: User has access to all locations in chain
  - SINGLE: User has access to specific location_id only
```

### 4.3 Access Resolution Logic

```java
List<LocationInfo> resolveAccessibleLocations(userId, chainId) {
    UserAccess access = getUserAccess(userId, chainId);

    if (access.type == ALL) {
        return getAllChainLocations(chainId);
    } else {
        return getSpecificLocations(access.locationIds);
    }
}
```

---

## 5. Partner Integration Interface

### 5.1 Generic Partner Contract

```java
/**
 * Generic interface for partner integrations.
 * Partners (PIE, DoorDash, Uber, etc.) implement specific adapters.
 */
public interface PartnerIntegration {

    /**
     * Get user's accessible locations for this partner.
     */
    List<PartnerLocation> getAccessibleLocations(Long userId);

    /**
     * Generate SSO/access token for partner system.
     */
    PartnerToken generateToken(PartnerTokenRequest request);

    /**
     * Map Chowbus location to partner location ID.
     */
    String mapToPartnerLocationId(Long chowbusLocationId);

    /**
     * Map Chowbus chain to partner entity ID.
     */
    String mapToPartnerId(Long chowbusChainId);
}
```

### 5.2 Structure Mapping (结构对映)

```
┌─────────────────────────────────────────────────────────┐
│                    STRUCTURE MAPPING                     │
├────────────────────────┬────────────────────────────────┤
│       CHOWBUS          │           PARTNER              │
├────────────────────────┼────────────────────────────────┤
│  Chain (Brand)         │  partner_id                    │
│  Restaurant            │  partner_location_id           │
│  User                  │  partner_user_id (optional)    │
│  Role                  │  partner_role (mapped)         │
├────────────────────────┼────────────────────────────────┤
│                   EXAMPLE (PIE)                         │
├────────────────────────┼────────────────────────────────┤
│  Chain ID: 9999        │  partner_id: "9999"            │
│  Restaurant ID: 12345  │  partner_location_id: "12345"  │
└────────────────────────┴────────────────────────────────┘
```

### 5.3 Mapping Strategies

| Strategy | Description | Use Case |
|----------|-------------|----------|
| Direct | Use Chowbus ID as partner ID | PIE (current design) |
| Registered | External registration required | Partners with own ID system |
| Computed | Algorithm-based mapping | Legacy integrations |

---

## 6. Solution Approaches

### 6.1 Solution A: Location Selection Flow

**Description**: User explicitly selects which locations to access before generating token.

```
┌─────────┐     ┌─────────────┐     ┌──────────────┐     ┌─────────┐
│  User   │────▶│ /locations  │────▶│ Select 1-N   │────▶│  Token  │
│ Request │     │ (list)      │     │ Locations    │     │ w/scope │
└─────────┘     └─────────────┘     └──────────────┘     └─────────┘
```

**Pros:**
- Explicit user control
- Clear audit trail
- Flexible scope selection
- Works for any number of locations

**Cons:**
- Extra API call required
- UI development needed for selection
- Slightly longer user flow

### 6.2 Solution B: Full-Access Token

**Description**: Token automatically includes all user's accessible locations.

```
┌─────────┐     ┌─────────────────┐     ┌───────────────────┐
│  User   │────▶│ /sso_token      │────▶│ Token w/all       │
│ Request │     │ (auto-resolve)  │     │ accessible locs   │
└─────────┘     └─────────────────┘     └───────────────────┘
```

**Pros:**
- Simple user flow (one API call)
- No selection needed
- Faster SSO experience

**Cons:**
- No granular control
- May expose more data than needed
- Token could be very large for chains with many locations

### 6.3 Solution C: Role-Based Scoping

**Description**: Token scope determined entirely by user's role.

```
Role = Boss     → ALL locations
Role = Manager  → Assigned store only
Role = Marketing→ Marketing-designated locations
```

**Pros:**
- Predictable behavior
- Easy to understand
- Minimal configuration

**Cons:**
- Inflexible (same role always = same scope)
- Doesn't support per-user location customization
- May not match business requirements

### 6.4 Solution D: Hybrid Approach (Recommended)

**Description**: Combine auto-resolution with optional explicit selection.

```
┌─────────────────────────────────────────────────────────┐
│                    HYBRID FLOW                           │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  IF user has 1 location:                                │
│    → Direct to /sso_token (single location, fast)       │
│                                                          │
│  ELSE IF user has multiple locations:                   │
│    → GET /locations first                               │
│    → User selects desired scope                         │
│    → POST /multi_location/sso_token                     │
│                                                          │
│  Optional: "Remember my selection" for repeat access    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

**Pros:**
- Best of both worlds
- Fast path for single-store users
- Flexible for chain users
- Backward compatible

**Cons:**
- More complex implementation
- Need to handle both paths

### 6.5 Comparison Matrix

| Criteria | Solution A | Solution B | Solution C | Solution D |
|----------|------------|------------|------------|------------|
| User Control | ★★★ | ★ | ★ | ★★★ |
| Simplicity | ★★ | ★★★ | ★★★ | ★★ |
| Flexibility | ★★★ | ★ | ★ | ★★★ |
| Performance | ★★ | ★★★ | ★★★ | ★★★ |
| Backward Compat | ★★★ | ★★ | ★ | ★★★ |
| **Total** | **13** | **10** | **9** | **14** |

**Decision**: Implement **Solution D (Hybrid Approach)**

---

## 7. Architecture Diagrams

### 7.1 Component Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                           NEXUS SERVICE                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    CONTROLLER LAYER                          │    │
│  │  ┌─────────────────┐  ┌─────────────────────────────────┐   │    │
│  │  │PieAuthController│  │ (Future Partner Controllers)    │   │    │
│  │  │ • /sso_token    │  │                                 │   │    │
│  │  │ • /locations    │  │                                 │   │    │
│  │  │ • /multi_loc..  │  │                                 │   │    │
│  │  └────────┬────────┘  └─────────────────────────────────┘   │    │
│  └───────────┼──────────────────────────────────────────────────┘    │
│              │                                                       │
│  ┌───────────▼──────────────────────────────────────────────────┐   │
│  │                     SERVICE LAYER                             │   │
│  │  ┌─────────────────┐  ┌─────────────────────────────────┐    │   │
│  │  │   PieService    │  │   ChowbusChainService           │    │   │
│  │  │ • Token Gen     │  │   • Chain lookup                │    │   │
│  │  │ • Location list │  │   • Location access             │    │   │
│  │  └────────┬────────┘  └───────────────┬─────────────────┘    │   │
│  │           │                           │                       │   │
│  │  ┌────────▼───────────────────────────▼─────────────────┐    │   │
│  │  │              TOKEN STRATEGY LAYER                     │    │   │
│  │  │  ┌─────────────────┐  ┌─────────────────────────┐    │    │   │
│  │  │  │SingleTokenStrat │  │MultiTokenStrategy       │    │    │   │
│  │  │  │(location array) │  │(token per location)     │    │    │   │
│  │  │  └─────────────────┘  └─────────────────────────┘    │    │   │
│  │  └──────────────────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                   INTEGRATION LAYER                           │   │
│  │  ┌─────────────────┐  ┌─────────────────────────────────┐    │   │
│  │  │ChowbusRestService│  │ChowbusSalesService             │    │   │
│  │  │(POS API)         │  │(Saleshub API)                  │    │   │
│  │  └─────────────────┘  └─────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       EXTERNAL SYSTEMS                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────┐  │
│  │   POS API   │  │  Saleshub   │  │  Chain/Location Service     │  │
│  │             │  │   Service   │  │  (TBD - internal API)       │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.2 Data Flow Diagram

```
┌────────────────────────────────────────────────────────────────────┐
│               MULTI-LOCATION SSO DATA FLOW                          │
└────────────────────────────────────────────────────────────────────┘

  ┌──────────┐
  │  Client  │
  │  (App)   │
  └────┬─────┘
       │
       │ 1. GET /locations
       ▼
  ┌────────────────────┐
  │  PieAuthController │
  └─────────┬──────────┘
            │
            │ 2. Get user context
            ▼
  ┌────────────────────┐     3. Get chain info     ┌──────────────────┐
  │    PieService      │ ─────────────────────────▶│ ChowbusChainSvc  │
  └─────────┬──────────┘                           └────────┬─────────┘
            │                                               │
            │                                    4. Query   │
            │                                     internal  │
            │                                      APIs     ▼
            │                                     ┌──────────────────┐
            │                                     │  Internal APIs   │
            │                                     │ • Chain info     │
            │                                     │ • User locations │
            │                                     └──────────────────┘
            │
            │ 5. Return location list
            ▼
  ┌────────────────────┐
  │     Response       │
  │  {locations: [...]}│
  └────────────────────┘

       │
       │ 6. POST /multi_location/sso_token
       │    {locationIds: [...]}
       ▼
  ┌────────────────────┐
  │  PieAuthController │
  └─────────┬──────────┘
            │
            │ 7. Validate access & generate token
            ▼
  ┌────────────────────┐     8. Get merchant     ┌──────────────────┐
  │    PieService      │ ────────────info────────▶│ ChowbusRestSvc   │
  └─────────┬──────────┘                         └──────────────────┘
            │
            │ 9. Build JWT with multi-location claims
            ▼
  ┌────────────────────┐
  │  TokenStrategy     │
  │  (Single/Multi)    │
  └─────────┬──────────┘
            │
            │ 10. Return token(s)
            ▼
  ┌────────────────────┐
  │     Response       │
  │  {token: "...",    │
  │   scope: "MULTI"}  │
  └────────────────────┘
```

---

## 8. User Flow Diagrams

### 8.1 Single-Store User → Partner (Unchanged)

```
┌─────────────────────────────────────────────────────────────┐
│             SINGLE-STORE USER FLOW (backward compatible)     │
└─────────────────────────────────────────────────────────────┘

  ┌──────┐                ┌───────┐                 ┌─────┐
  │ User │                │ Nexus │                 │ PIE │
  └──┬───┘                └───┬───┘                 └──┬──┘
     │                        │                        │
     │  1. Click "Advertising"│                        │
     │────────────────────────▶                        │
     │                        │                        │
     │                  [Check: single location]       │
     │                        │                        │
     │  2. POST /sso_token    │                        │
     │────────────────────────▶                        │
     │                        │                        │
     │  3. Token (single loc) │                        │
     │◀────────────────────────                        │
     │                        │                        │
     │  4. Redirect with token│                        │
     │─────────────────────────────────────────────────▶
     │                        │                        │
     │  5. PIE Dashboard      │                        │
     │◀─────────────────────────────────────────────────
     │     (single store)     │                        │
```

### 8.2 Chain HQ User → Partner (Multi-Location)

```
┌─────────────────────────────────────────────────────────────┐
│                 CHAIN HQ USER FLOW                           │
└─────────────────────────────────────────────────────────────┘

  ┌──────┐                ┌───────┐                 ┌─────┐
  │ User │                │ Nexus │                 │ PIE │
  └──┬───┘                └───┬───┘                 └──┬──┘
     │                        │                        │
     │  1. Click "Advertising"│                        │
     │────────────────────────▶                        │
     │                        │                        │
     │                  [Check: chain user,            │
     │                   multiple locations]           │
     │                        │                        │
     │  2. GET /locations     │                        │
     │────────────────────────▶                        │
     │                        │                        │
     │  3. Location list      │                        │
     │◀────────────────────────                        │
     │    [{id:12345,name:"NYC"},                      │
     │     {id:12346,name:"LA"},...]                   │
     │                        │                        │
     │  4. [User selects locations]                    │
     │                        │                        │
     │  5. POST /multi_location/sso_token              │
     │     {locationIds: [12345,12346]}                │
     │────────────────────────▶                        │
     │                        │                        │
     │  6. Token (multi loc)  │                        │
     │◀────────────────────────                        │
     │                        │                        │
     │  7. Redirect with token│                        │
     │─────────────────────────────────────────────────▶
     │                        │                        │
     │  8. PIE Dashboard      │                        │
     │◀─────────────────────────────────────────────────
     │    (multi-store view)  │                        │
```

### 8.3 Chain Store User → Partner (Single Location)

```
┌─────────────────────────────────────────────────────────────┐
│               CHAIN STORE USER FLOW                          │
└─────────────────────────────────────────────────────────────┘

  ┌──────┐                ┌───────┐                 ┌─────┐
  │ User │                │ Nexus │                 │ PIE │
  └──┬───┘                └───┬───┘                 └──┬──┘
     │                        │                        │
     │  1. Click "Advertising"│                        │
     │────────────────────────▶                        │
     │                        │                        │
     │                  [Check: chain user,            │
     │                   but only 1 location access]   │
     │                        │                        │
     │  2. POST /sso_token    │                        │
     │────────────────────────▶                        │
     │     (auto: single loc) │                        │
     │                        │                        │
     │  3. Token (single loc) │                        │
     │◀────────────────────────                        │
     │                        │                        │
     │  4. Redirect with token│                        │
     │─────────────────────────────────────────────────▶
     │                        │                        │
     │  5. PIE Dashboard      │                        │
     │◀─────────────────────────────────────────────────
     │    (single store view) │                        │
```

---

## 9. PIE Implementation (First Consumer)

### 9.1 PIE-Specific JWT Claims

**Single-location token (backward compatible):**
```json
{
  "sub": "user_123",
  "iss": "nexus.chowbus.com",
  "aud": "pie",
  "iat": 1705632000,
  "exp": 1705635600,
  "partner_id": "",
  "partner_location_id": "12345",
  "partner_user_id": "user_123",
  "partner_role": "marketing",
  "name": "John Doe",
  "email": "john@example.com"
}
```

**Multi-location token (new):**
```json
{
  "sub": "user_123",
  "iss": "nexus.chowbus.com",
  "aud": "pie",
  "iat": 1705632000,
  "exp": 1705635600,
  "partner_id": "99999",
  "partner_location_id": "12345",
  "partner_location_ids": ["12345", "12346", "12347"],
  "access_scope": "MULTI",
  "partner_user_id": "user_123",
  "partner_role": "marketing",
  "name": "John Doe",
  "email": "john@example.com"
}
```

### 9.2 Claim Definitions

| Claim | Type | Description |
|-------|------|-------------|
| `partner_id` | String | Chain ID (empty for single-store) |
| `partner_location_id` | String | Primary location ID (always present) |
| `partner_location_ids` | String[] | All accessible location IDs (multi-loc only) |
| `access_scope` | Enum | SINGLE, MULTI, or ALL |
| `partner_user_id` | String | User identifier for partner |
| `partner_role` | String | Mapped role for partner |

### 9.3 Access Scope Values

| Scope | Description | Use Case |
|-------|-------------|----------|
| `SINGLE` | One location only | Single-store user or store-level user |
| `MULTI` | Specific selected locations | HQ user with partial selection |
| `ALL` | All locations in chain | HQ user with full access |

### 9.4 PIE Redirect URL Format

```
{PIE_BASE_URL}/sso?token={jwt_token}

Example:
https://partner.pie.com/sso?token=eyJhbGciOiJSUzI1NiIs...
```

---

## 10. Implementation Plan

### 10.1 Phase Overview

| Phase | Module(s) | Deliverables | Dependencies |
|-------|-----------|--------------|--------------|
| 1 | nexus-client, nexus-common | DTOs, Constants, Enums | None |
| 2 | nexus-integration | ChowbusChainService | Internal API specs |
| 3 | nexus-service | Service logic, Token strategy | Phase 1, 2 |
| 4 | nexus-start | Controller endpoints | Phase 3 |
| 5 | Configuration | application.yml updates | Phase 4 |

### 10.2 Phase 1: Data Model (nexus-client, nexus-common)

**New DTOs:**

```java
// nexus-client/.../dto/LocationInfo.java
public class LocationInfo {
    private Long locationId;
    private String locationName;
    private String address;
    private Boolean hasAdvertisingContract;
}

// nexus-client/.../dto/ChainInfo.java
public class ChainInfo {
    private Long chainId;
    private String chainName;
    private List<LocationInfo> locations;
}

// nexus-client/.../dto/UserLocationAccessResponse.java
public class UserLocationAccessResponse {
    private Long chainId;
    private String chainName;
    private UserPerspective perspective
 // HQ or STORE
    private List<LocationInfo> accessibleLocations;
}

// nexus-client/.../dto/MultiLocationSsoRequest.java
public class MultiLocationSsoRequest {
    private Long chainId;
    private List<Long> locationIds;
}

// nexus-client/.../dto/MultiLocationSsoTokenResponse.java
public class MultiLocationSsoTokenResponse {
    private String token;
    private String redirectUrl;
    private AccessScope scope;
    private Long primaryLocationId;
    private List<Long> locationIds;
}
```

**New Constants:**

```java
// nexus-common/.../constant/AccessScope.java
public enum AccessScope {
    SINGLE,   // One location only
    MULTI,    // Multiple specific locations
    ALL       // All locations in chain
}

// nexus-common/.../constant/UserPerspective.java
public enum UserPerspective {
    HQ,       // Headquarters/brand level
    STORE     // Single store level
}
```

**New Error Codes:**

```java
// Add to nexus-common/.../constant/ErrorCode.java
LOCATION_ACCESS_DENIED(310001, "User does not have access to requested location"),
CHAIN_NOT_FOUND(310002, "Chain not found for restaurant"),
NO_ACCESSIBLE_LOCATIONS(310003, "User has no accessible locations")
```

### 10.3 Phase 2: Integration Layer (nexus-integration)

**New Service:**

```java
// nexus-integration/.../service/ChowbusChainService.java
@Service
public class ChowbusChainService {

    /**
     * Get chain info for a restaurant (if it belongs to a chain).
     * Returns null if restaurant is single-store.
     */
    ChainInfo getChainInfo(Long restaurantId);

    /**
     * Get user's accessible locations within a chain.
     * Considers user's role and jurisdiction scope.
     */
    List<LocationInfo> getUserAccessibleLocations(Long userId, Long chainId);

    /**
     * Batch check if user has advertising permissions for locations.
     */
    Map<Long, Boolean> checkAdvertisingPermissions(Long userId, List<Long> locationIds);

    /**
     * Get user's role and perspective (HQ vs Store).
     */
    UserRoleInfo getUserRoleInfo(Long userId, Long restaurantId);
}
```

**Required Internal APIs (to be confirmed):**

| API | Method | Purpose |
|-----|--------|---------|
| `/api/internal/v1/restaurants/{id}/chain` | GET | Get chain info |
| `/api/internal/v1/chains/{chainId}/locations` | GET | List chain locations |
| `/api/internal/v1/users/{userId}/accessible_locations` | GET | User's accessible locations |
| `/api/internal/v1/users/{userId}/role` | GET | User role info |

### 10.4 Phase 3: Service Layer (nexus-service)

**Extend PieService interface:**

```java
// Add to nexus-client/.../api/PieService.java
UserLocationAccessResponse getUserAccessibleLocations();
MultiLocationSsoTokenResponse generateMultiLocationSsoToken(MultiLocationSsoRequest request);
```

**Token Strategy Pattern:**

```java
// nexus-service/.../strategy/PieTokenStrategy.java
public interface PieTokenStrategy {
    MultiLocationSsoTokenResponse generateToken(
        MultiLocationSsoRequest request,
        List<MerchantInfo> merchants
    );
}

// nexus-service/.../strategy/SingleTokenStrategy.java
@Component
public class SingleTokenStrategy implements PieTokenStrategy {
    // Returns ONE token with partner_location_ids array
}

// nexus-service/.../strategy/MultiTokenStrategy.java
@Component
public class MultiTokenStrategy implements PieTokenStrategy {
    // Returns MULTIPLE tokens, one per location
}
```

### 10.5 Phase 4: Controller Layer (nexus-start)

**Extend PieAuthController:**

```java
// nexus-start/.../controller/PieAuthController.java

/**
 * Get user's accessible locations for advertising.
 * Returns location list for multi-location selection UI.
 */
@GetMapping("/locations")
public ResponseEntity<UserLocationAccessResponse> getAccessibleLocations() {
    return ResponseEntity.ok(pieService.getUserAccessibleLocations());
}

/**
 * Generate multi-location SSO token.
 * Used when user has selected multiple locations.
 */
@PostMapping("/multi_location/sso_token")
public ResponseEntity<MultiLocationSsoTokenResponse> generateMultiLocationToken(
    @RequestBody @Valid MultiLocationSsoRequest request
) {
    return ResponseEntity.ok(pieService.generateMultiLocationSsoToken(request));
}
```

### 10.6 Phase 5: Configuration

```yaml
# Add to nexus-start/src/main/resources/application.yml
pie:
  multi-location:
    enabled: true
    max-locations-per-request: 50
    token-strategy: single  # single or multi
```

### 10.7 Files to Modify Summary

| File | Changes |
|------|---------|
| `nexus-client/.../dto/LocationInfo.java` | **New** |
| `nexus-client/.../dto/ChainInfo.java` | **New** |
| `nexus-client/.../dto/UserLocationAccessResponse.java` | **New** |
| `nexus-client/.../dto/MultiLocationSsoRequest.java` | **New** |
| `nexus-client/.../dto/MultiLocationSsoTokenResponse.java` | **New** |
| `nexus-common/.../constant/AccessScope.java` | **New** |
| `nexus-common/.../constant/UserPerspective.java` | **New** |
| `nexus-common/.../constant/ErrorCode.java` | Add 3 error codes |
| `nexus-integration/.../dto/MerchantInfo.java` | Add chain fields |
| `nexus-integration/.../service/ChowbusChainService.java` | **New** |
| `nexus-client/.../api/PieService.java` | Add 2 methods |
| `nexus-service/.../service/PieServiceImpl.java` | Implement multi-loc |
| `nexus-service/.../strategy/PieTokenStrategy.java` | **New** |
| `nexus-service/.../strategy/SingleTokenStrategy.java` | **New** |
| `nexus-service/.../strategy/MultiTokenStrategy.java` | **New** |
| `nexus-start/.../controller/PieAuthController.java` | Add 2 endpoints |
| `nexus-start/.../resources/application.yml` | Add config |

---

## 11. Dependencies & Open Questions

### 11.1 Prerequisites Before Implementation

| Dependency | Owner | Status | Blocker? |
|------------|-------|--------|----------|
| PIE token format confirmation | PIE Team | Pending | No (abstraction layer handles both) |
| Internal APIs for chain data | Backend Team | Pending | Yes (Phase 2) |
| Chain/location data schema | Data Team | Pending | Partial |

### 11.2 Required Internal API Clarification

**Action Item**: Confirm which APIs exist and their specifications:

| API | Purpose | Exists? |
|-----|---------|---------|
| `GET /api/internal/v1/restaurants/{id}/chain` | Chain info lookup | TBD |
| `GET /api/internal/v1/chains/{chainId}/locations` | Chain locations | TBD |
| `GET /api/internal/v1/users/{userId}/accessible_locations` | User's location access | TBD |
| `GET /api/internal/v1/users/{userId}/role` | User role info | TBD |

### 11.3 Open Questions

| # | Question | Status | Impact |
|---|----------|--------|--------|
| 1 | PIE token format: single token with array vs multiple tokens? | Pending PIE confirmation | Token strategy selection |
| 2 | Should all selected locations require advertising contracts? | Pending business decision | Validation logic |
| 3 | Token expiration for multi-location tokens? | Pending | JWT configuration |
| 4 | Rate limiting for /locations endpoint? | Pending | Performance |

### 11.4 Can Start Now

Even with pending items, these can begin:
- Phase 1: All DTOs and constants (no external dependencies)
- Phase 3 partial: Service interfaces and strategy pattern
- Phase 4 partial: Controller structure

### 11.5 Blocked Until API Confirmation

- Phase 2: Integration layer (needs API specs)
- Phase 3 complete: Full service implementation

---

## Appendix A: API Specification

### A.1 GET /api/v1/advertising/locations

**Purpose**: Get user's accessible locations for advertising

**Request:**
```http
GET /api/v1/advertising/locations
Authorization: Bearer <jwt>
```

**Response (Chain User):**
```json
{
  "chainId": 99999,
  "chainName": "Happy Dumpling",
  "perspective": "HQ",
  "accessibleLocations": [
    {
      "locationId": 12345,
      "locationName": "Happy Dumpling - NYC",
      "address": "123 Main St, New York, NY",
      "hasAdvertisingContract": true
    },
    {
      "locationId": 12346,
      "locationName": "Happy Dumpling - LA",
      "address": "456 Oak Ave, Los Angeles, CA",
      "hasAdvertisingContract": true
    }
  ]
}
```

**Response (Single-Store User):**
```json
{
  "chainId": null,
  "chainName": null,
  "perspective": "STORE",
  "accessibleLocations": [
    {
      "locationId": 12345,
      "locationName": "Joe's Diner",
      "address": "789 Elm St, Chicago, IL",
      "hasAdvertisingContract": true
    }
  ]
}
```

### A.2 POST /api/v1/advertising/multi_location/sso_token

**Purpose**: Generate SSO token for multiple locations

**Request:**
```http
POST /api/v1/advertising/multi_location/sso_token
Authorization: Bearer <jwt>
Content-Type: application/json

{
  "chainId": 99999,
  "locationIds": [12345, 12346]
}
```

**Response:**
```json
{
  "token": "eyJhbGciOiJSUzI1NiIs...",
  "redirectUrl": "https://partner.pie.com/sso?token=eyJhbGciOiJSUzI1NiIs...",
  "scope": "MULTI",
  "primaryLocationId": 12345,
  "locationIds": [12345, 12346]
}
```

**Error Responses:**

| Code | Status | Description |
|------|--------|-------------|
| 310001 | 403 | Location access denied |
| 310002 | 404 | Chain not found |
| 310003 | 400 | No accessible locations |

---

## Appendix B: Verification Plan

### B.1 Unit Tests

```java
// Test single-store user
@Test
void getUserAccessibleLocations_singleStoreUser_returnsSingleLocation() {
    // Given: User with single-store tenant
    // When: getUserAccessibleLocations()
    // Then: Returns 1 location, perspective=STORE
}

// Test chain HQ user
@Test
void getUserAccessibleLocations_chainHqUser_returnsMultipleLocations() {
    // Given: User with HQ role in chain
    // When: getUserAccessibleLocations()
    // Then: Returns all accessible locations, perspective=HQ
}

// Test multi-location token
@Test
void generateMultiLocationToken_validRequest_returnsTokenWithAllLocations() {
    // Given: Valid request with accessible location IDs
    // When: generateMultiLocationSsoToken()
    // Then: Token contains partner_location_ids array
}

// Test access denied
@Test
void generateMultiLocationToken_unauthorizedLocation_throwsAccessDenied() {
    // Given: Request with location user cannot access
    // When: generateMultiLocationSsoToken()
    // Then: Throws LOCATION_ACCESS_DENIED error
}
```

### B.2 Integration Tests

```java
@Test
void fullFlow_singleStoreUser_backwardCompatible() {
    // Verify existing /sso_token endpoint still works
}

@Test
void fullFlow_chainHqUser_multiLocationFlow() {
    // 1. GET /locations -> returns multiple
    // 2. POST /multi_location/sso_token -> returns multi-loc token
    // 3. Verify JWT claims
}

@Test
void fullFlow_chainStoreUser_singleLocationOnly() {
    // Chain user with single location access
    // Should behave like single-store user
}
```

### B.3 Manual Testing

```bash
# Get accessible locations
curl -X GET http://localhost:9001/api/v1/advertising/locations \
  -H "Authorization: Bearer <jwt>"

# Generate multi-location token
curl -X POST http://localhost:9001/api/v1/advertising/multi_location/sso_token \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -d '{"chainId": 99999, "locationIds": [12345, 12346]}'
```

---

*Document Version: 1.0*
*Last Updated: 2026-01-19*
