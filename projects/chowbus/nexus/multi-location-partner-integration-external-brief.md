# Multi-Location Account Management for Partner Integrations

**External Stakeholder Brief**

> **Document Purpose**: Communication with external partners and business stakeholders
> **Version**: 1.0
> **Date**: 2026-01-19
> **Audience**: Partner integration teams, product managers, business stakeholders

---

## Executive Summary

Chowbus is expanding its partner integration capabilities to support **chain and franchise restaurant structures**. This enhancement enables restaurant groups with multiple locations to manage their external partner accounts (advertising, delivery, etc.) across all their stores through a unified experience.

**Key Benefits:**
- Chain restaurants can manage multiple locations from a single account
- Flexible access control per user (full chain access or specific stores only)
- Backward compatible - existing single-store integrations unchanged
- First partner: PIE advertising platform

---

## Business Context

### Current State

Today, Chowbus partner integrations support **single-location access only**:
- One user → One restaurant → One partner account
- Chain restaurants must manage each location separately
- No unified view across multiple stores

### Future State

The enhanced integration supports **multi-location access**:
- One user → Multiple restaurants → Unified partner experience
- HQ users can access all stores or a subset
- Store managers access only their assigned location(s)

---

## User Scenarios

### Scenario 1: Single-Store Restaurant (Unchanged)

```
Restaurant Owner (Joe's Diner)
    │
    └──► Partner Platform (PIE)
         └── Single store dashboard
```

**Experience**: No change. User clicks "Advertising" and goes directly to PIE with single-store access.

### Scenario 2: Chain Restaurant - HQ User

```
Marketing Manager (Happy Dumpling HQ)
    │
    ├──► Select stores: NYC, LA, Chicago
    │
    └──► Partner Platform (PIE)
         └── Multi-store dashboard (selected stores)
```

**Experience**: User sees a location selector, chooses which stores to access, then proceeds to PIE with multi-location view.

### Scenario 3: Chain Restaurant - Store Manager

```
Store Manager (Happy Dumpling - NYC)
    │
    └──► Partner Platform (PIE)
         └── Single store dashboard (NYC only)
```

**Experience**: Similar to single-store. User has access to only their assigned location.

---

## Chain Structure Overview

### Tenant Types

| Type | Description | Partner Access |
|------|-------------|----------------|
| **Single-Store** | Independent restaurant | One location only |
| **Chain** | Brand with multiple locations | One or more locations based on user's role |

### User Perspectives

| Perspective | Description | Typical Users |
|-------------|-------------|---------------|
| **HQ (总部)** | Headquarters/brand level | Boss, Marketing, Operations, IT |
| **Store (门店)** | Individual store level | Store Manager, Staff |

### Access Control Model

Users in a chain can have different location scopes:

| User | Role | Accessible Locations |
|------|------|---------------------|
| Alice | Marketing (HQ) | NYC, LA, Chicago |
| Bob | Marketing (HQ) | Houston, Seattle |
| Carol | Boss (HQ) | All locations |
| Dave | Store Manager | NYC only |

---

## Partner Integration Requirements

### What Partners Receive

When a user authenticates via SSO, partners receive a JWT token containing:

**Single-Location Token:**
```
partner_id: ""                    (empty for single-store)
partner_location_id: "12345"      (restaurant ID)
```

**Multi-Location Token:**
```
partner_id: "99999"               (chain ID)
partner_location_id: "12345"      (primary location for backward compat)
partner_location_ids: ["12345", "12346", "12347"]
access_scope: "MULTI"             (SINGLE | MULTI | ALL)
```

### Partner Action Items

| Item | Description | Required? |
|------|-------------|-----------|
| Support `partner_location_ids` array | Parse and use location list from token | Yes |
| Handle `access_scope` claim | Understand user's access level | Recommended |
| Multi-location dashboard | Show data for multiple locations | Recommended |
| Backward compatibility | Continue supporting single-location tokens | Yes |

### Token Format Discussion

**Open Question for Partners:**

We're designing flexibility for how multi-location access is communicated:

| Option | Description | Partner Impact |
|--------|-------------|----------------|
| **Option A** | Single token with location array | Parse array, build multi-location view |
| **Option B** | Multiple tokens (one per location) | Manage multiple sessions or combine data |

Please indicate your preference and any technical constraints.

---

## Timeline & Phases

### Phase 1: Foundation (In Progress)
- Data models and interfaces
- No partner-visible changes

### Phase 2: Integration Layer
- Internal API connections for chain/location data
- Backend testing

### Phase 3: Partner Integration
- New SSO endpoints available
- Partner testing and validation
- **Partner involvement required**

### Phase 4: Rollout
- Gradual rollout to chain restaurants
- Monitoring and support

---

## Frequently Asked Questions

### General

**Q: Will this break existing integrations?**
A: No. Single-store flow remains unchanged. Existing tokens continue to work.

**Q: When will this be available?**
A: Timeline TBD pending internal API readiness and partner confirmation.

**Q: Which partners will support this first?**
A: PIE advertising platform is the first integration. Others will follow.

### Technical

**Q: How do we identify chain vs single-store users?**
A: Check if `partner_id` is populated and/or if `partner_location_ids` array exists.

**Q: What if a location leaves or joins a chain?**
A: Tokens are generated fresh each session. Changes are reflected on next login.

**Q: How long are tokens valid?**
A: Tokens expire after 1 hour. Users re-authenticate to get updated access.

**Q: Can users change their location selection mid-session?**
A: They would need to return to Chowbus and re-initiate SSO with new selection.

### Business

**Q: How are locations matched to partner accounts?**
A: We use Chowbus restaurant IDs as `partner_location_id`. Partners map these to their internal IDs.

**Q: What about franchises vs corporate-owned chains?**
A: Both are supported. The access control model handles any ownership structure.

**Q: Can a user belong to multiple chains?**
A: Currently, a user session is associated with one chain at a time.

---

## Contact & Next Steps

### For Partner Integration Teams

1. **Review** this document and the token format options
2. **Confirm** your preferred token format (Option A or B)
3. **Identify** any technical constraints or requirements
4. **Schedule** a technical sync to align on implementation

### Points of Contact

| Role | Contact |
|------|---------|
| Product Manager | [TBD] |
| Engineering Lead | [TBD] |
| Partner Integration | [TBD] |

---

## Appendix: Glossary

| Term | Definition |
|------|------------|
| **Chain** | A restaurant brand with multiple locations (e.g., "Happy Dumpling" with stores in NYC, LA, Chicago) |
| **Location** | A single physical restaurant/store |
| **HQ** | Headquarters - the central management of a chain |
| **Partner** | External service integrated via SSO (e.g., PIE advertising platform) |
| **SSO** | Single Sign-On - authentication method allowing Chowbus users to access partner platforms |
| **JWT** | JSON Web Token - secure token format used for SSO authentication |
| **Access Scope** | The set of locations a user can access (SINGLE, MULTI, or ALL) |
| **Perspective** | User's view level - HQ (sees multiple stores) or STORE (sees one store) |

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-19 | Engineering | Initial draft for partner review |

---

*This document is for external stakeholder communication. For technical implementation details, see the internal design documentation.*
