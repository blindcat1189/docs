# Result Model Design for Cross-Service Compatibility

**Date:** 2025-01-30
**Status:** Implemented
**Module:** chowbus-boot-base

## Context

The Chowbus ecosystem has multiple services in different languages (Go, Java, Ruby) with inconsistent API response formats. This creates challenges for:
- Frontend/mobile clients handling different response structures
- Inter-service communication
- Error handling consistency

### Current State Analysis

| Service | Language | Success Field | Error Code Field | Pagination Location |
|---------|----------|---------------|------------------|---------------------|
| Proto (canonical) | N/A | Implicit | `code` (string) | N/A |
| Catalog | Go | Implicit | `code` (from Type) | `page` object |
| POS/Harbor | Go | Implicit | `code` (int) + `type` (string) | `meta` object |
| Ruby Monolith | Ruby | HTTP status | None (title only) | `meta.pages` |
| chowbus-boot | Java | `success` boolean | `type` (string) | `page` object |

### Key Issues Identified

1. **Field naming conflict**: Java used `type` while proto/Go use `code`
2. **Legacy Go services**: Have `Code int` (deprecated) and `Type string`
3. **Inconsistent pagination**: `page` vs `meta` vs `meta.pages`
4. **Ruby services**: Don't return error codes at all

## Decision

### 1. Error Field Naming: `code` (string)

Align with proto definition. Java field renamed from `type` to `code`.

```java
// ChowbusError.java
@JSONField(name = "code", alternateNames = {"type"})
private String code;  // e.g., "Auth.Token.Invalid"
```

**Backward compatibility:**
- `@JSONField(alternateNames = {"type"})` accepts both in JSON input
- Deprecated `getType()`/`setType()` methods delegate to `code`
- Deprecated `.type()` builder method

### 2. Pagination Location: `page` object (not `meta`)

Keep pagination in dedicated `page` object, separate from `meta`.

**Rationale:**
- `page` is specific and well-defined
- `meta` is generic (can contain anything)
- Aligns with Catalog (cleanest Go service) and current Java
- Industry patterns (HAL, Spring Data) use separate pagination objects

```json
{
  "data": [...],
  "page": {
    "page": 1,
    "page_size": 20,
    "total": 100,
    "sort_field": "created_at",
    "sort_direction": "desc"
  },
  "meta": {
    "request_id": "abc-123"
  }
}
```

### 3. Response Meta: Separate from Error Meta

Two distinct `meta` concepts:
- **Error meta** (`errors[].meta`): Error-specific details (field errors, retry hints)
- **Response meta** (`meta`): Response context (request_id, currency, timezone)

### 4. JSON Naming: snake_case (opt-in)

Use `SnakeCaseJsonAutoConfiguration` for automatic conversion:

```yaml
chowbus:
  json:
    snake-case-mvc-enabled: true
```

When enabled, all camelCase fields become snake_case in JSON output.

### 5. Keep `success` Field

Java services keep explicit `success` boolean for:
- Dubbo RPC clarity between Java services
- Backward compatibility with existing consumers
- Explicit contract (proto defines error structure, not full envelope)

## Final Response Structure

### SingleResult (single item)
```json
{
  "success": true,
  "data": { "id": 1, "name": "Item" },
  "errors": null,
  "meta": { "request_id": "abc-123" }
}
```

### MultiResult (paginated list)
```json
{
  "success": true,
  "data": [...],
  "page": {
    "page": 1,
    "page_size": 20,
    "total": 100,
    "sort_field": "created_at",
    "sort_direction": "desc"
  },
  "errors": null,
  "meta": { "request_id": "abc-123", "currency": "USD" }
}
```

### Error Response
```json
{
  "success": false,
  "data": null,
  "errors": [{
    "code": "Auth.Token.Invalid",
    "title": "Please log in again",
    "detail": "JWT token has expired",
    "meta": { "retry_after": "3600" }
  }],
  "meta": { "request_id": "abc-123" }
}
```

## Implementation

### Files Changed

| File | Changes |
|------|---------|
| `ChowbusError.java` | Rename `type`→`code`, add compatibility layer |
| `Result.java` | Add `meta` field with `withMeta()` methods |
| `SingleResult.java` | Override `withMeta()` for fluent chaining |
| `MultiResult.java` | Override `withMeta()` for fluent chaining |
| `Page.java` | Add `sortField`, `sortDirection` |
| `ResultWrapper.java` | Update `.type()` → `.code()` |
| `GlobalExceptionHandler.java` | Update `.type()` → `.code()` |

### Usage Examples

```java
// Single item with meta
return SingleResult.buildSuccess(user)
    .withMeta("request_id", requestId);

// Paginated list with sorting
Page page = Page.of(1, 20, 100, "created_at", "desc");
return MultiResult.buildSuccess(items, page)
    .withMeta("currency", "USD");

// Error with meta
return SingleResult.buildFailure("Auth.Token.Invalid", "Please log in")
    .withMeta("request_id", requestId);
```

### Configuration

```yaml
# application.yml
chowbus:
  json:
    snake-case-mvc-enabled: true  # Required for Go compatibility
```

## Migration Path

### Phase 1: Java (Done)
- Rename `type` → `code` with compatibility layer
- Add `meta` to Result classes
- Add `sortField`/`sortDirection` to Page

### Phase 2: Go Services (Future)
- Migrate from `meta` to `page` for pagination
- Deprecate `Code int`, use `Type string` → JSON `code`

### Phase 3: Ruby Services (Future)
- Add error codes to API responses
- Align with proto Error structure

## References

- [Proto Definition: error_resp.proto](/Users/kado/chowbus/idl/definitions/chowbus/v1/error_resp.proto)
- [JSON:API Specification](https://jsonapi.org/format/)
- [HAL Specification](https://www.apigility.org/documentation/api-primer/halprimer)
- [Go Catalog response_handler.go](/Users/kado/chowbus/catalog/internal/controller/context/response_handler.go)
