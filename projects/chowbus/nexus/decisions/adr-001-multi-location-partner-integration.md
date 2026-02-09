# ADR-001: Multi-Location Partner Integration Architecture

**Date**: 2026-01-19
**Status**: Proposed
**Deciders**: Engineering Team
**Technical Story**: Multi-Location Account Management for Partner Integrations

---

## Context

Nexus currently handles single-location authentication for partner integrations (specifically PIE SSO). The existing implementation:

- Uses `partner_location_id` to identify a single restaurant
- Leaves `partner_id` empty/unused
- Has no support for chain/franchise tenant structures
- Cannot handle per-user location access control

Chowbus needs to support chain restaurants where:
1. A single brand has multiple locations
2. HQ users may need access to multiple/all locations
3. Store-level users should only access their assigned location(s)
4. External partners need consistent mapping to Chowbus's structure

## Decision

We will implement a **hybrid approach** for multi-location partner integration that:

1. **Maintains backward compatibility** - Existing single-location flow unchanged
2. **Adds new endpoints** for multi-location scenarios:
   - `GET /locations` - List user's accessible locations
   - `POST /multi_location/sso_token` - Generate multi-location token
3. **Uses a strategy pattern** for token generation to accommodate different partner requirements
4. **Maps Chowbus IDs directly** to partner IDs (no external registration needed)

### Architecture Decisions

#### Decision 1: Hybrid Flow vs Full-Access Token

**Chosen**: Hybrid Flow

We will use a hybrid approach where:
- Single-location users bypass location selection
- Multi-location users explicitly select locations

**Alternatives Considered**:
- **Full-Access Token**: Auto-include all accessible locations
  - Rejected: No user control, potentially large tokens
- **Role-Based Scoping**: Scope based purely on role
  - Rejected: Doesn't support per-user location customization

**Rationale**: Hybrid provides best UX for both user types while maintaining explicit control for chain users.

#### Decision 2: Partner ID Mapping Strategy

**Chosen**: Direct ID Mapping

Use Chowbus chain ID as `partner_id` and restaurant ID as `partner_location_id` directly.

**Alternatives Considered**:
- **External Registration**: Partner registers and assigns their own IDs
  - Rejected: Extra coordination, maintenance overhead
- **UUID Mapping**: Generate UUIDs and maintain mapping table
  - Rejected: Unnecessary complexity for current requirements

**Rationale**: Simpler implementation, no external dependencies, can always add mapping layer later if needed.

#### Decision 3: Token Format Flexibility

**Chosen**: Strategy Pattern with Configurable Implementation

Implement `PieTokenStrategy` interface with two implementations:
- `SingleTokenStrategy`: One token with `partner_location_ids` array
- `MultiTokenStrategy`: Multiple tokens, one per location

**Rationale**: PIE's exact requirements are TBD. This abstraction allows switching without code changes.

#### Decision 4: New Endpoints vs Extending Existing

**Chosen**: New Endpoints

Add new endpoints rather than extending existing `/sso_token`:
- `/api/v1/advertising/locations` (new)
- `/api/v1/advertising/multi_location/sso_token` (new)
- `/api/v1/advertising/sso_token` (unchanged)

**Alternatives Considered**:
- **Extend existing endpoint** with optional `locationIds` parameter
  - Rejected: Complicates existing contract, harder to version

**Rationale**: Cleaner separation, easier testing, true backward compatibility.

#### Decision 5: Access Control Model

**Chosen**: Per-User Location Access

User access determined by:
1. User's role (defines permissions)
2. User's jurisdiction scope (defines which locations)

This allows same role with different location access per user.

**Rationale**: Matches PM requirements (管辖范围 design) where Marketing Manager A can access locations 1-3 while Marketing Manager B accesses locations 4-5.

## Consequences

### Positive

- **Backward Compatible**: Existing integrations continue working
- **Flexible**: Supports both single and multi-location scenarios
- **Extensible**: Strategy pattern allows new token formats without core changes
- **Simple Mapping**: Direct ID usage avoids mapping maintenance
- **Clear Separation**: New endpoints for new functionality

### Negative

- **Additional Complexity**: More code paths to maintain
- **API Dependency**: Requires internal APIs for chain/location data (some TBD)
- **Testing Surface**: More scenarios to test (single, multi, chain vs non-chain)

### Risks

| Risk | Mitigation |
|------|------------|
| Internal APIs not ready | Can implement Phase 1 (DTOs) and Phase 4 (controller structure) first |
| PIE token format changes | Strategy pattern absorbs format changes |
| Performance with many locations | Add pagination, max locations config |

## Implementation

### Phase Summary

1. **Phase 1** (No dependencies): DTOs, constants, enums
2. **Phase 2** (Blocked on API specs): Integration layer
3. **Phase 3** (Depends on 1, 2): Service layer with strategy pattern
4. **Phase 4** (Depends on 3): Controller endpoints
5. **Phase 5** (Depends on 4): Configuration

### Key Files

| New | Modified |
|-----|----------|
| 5 DTOs in nexus-client | PieService interface |
| 2 enums in nexus-common | PieServiceImpl |
| ChowbusChainService | PieAuthController |
| PieTokenStrategy + impls | ErrorCode.java |
| | application.yml |

## Compliance

- **Security**: JWT tokens signed with RSA keys (existing mechanism)
- **Privacy**: Location access verified before token generation
- **Audit**: Token claims provide traceability

## References

- [Technical Design Document](../multi-location-partner-integration-design.md)
- [Summary Document](../multi-location-partner-integration-summary.md)
- PM Design: 角色权限, 管辖范围, 结构对映 diagrams

---

## Changelog

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-01-19 | Engineering | Initial draft |
