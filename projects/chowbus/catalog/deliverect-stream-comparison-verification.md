# Technical Verification Report: Deliverect vs Stream Platform Comparison

**Date:** 2026-01-19
**Verified By:** Code Analysis
**Repository:** catalog service

---

## Executive Summary

This report verifies the technical accuracy of the "Deliverect vs Stream Platform Comparison" document by cross-referencing claims with the actual codebase. **Overall Verdict: 98% Accurate** with minor omissions noted below.

---

## Verification Results

### ✅ 1. CDC Event Types and Triggers

**Document Claims:** 7 CDC event types trigger menu synchronization

**Verification Status:** ✅ **VERIFIED** (with addition)

**Source:** `internal/consumer/cdc_event_handler.go:106-127`

| # | Event Type | Document | Code | Notes |
|---|------------|----------|------|-------|
| 1 | Menu changes | ✅ | ✅ | Triggers menu publish + stat recalculation |
| 2 | Menu category changes | ✅ | ✅ | Triggers menu publish + stat recalculation |
| 3 | Category changes | ✅ | ✅ | Triggers menu publish for all affected menus |
| 4 | Meal instance changes | ✅ | ✅ | Triggers menu publish (marked as "most common") |
| 5 | Customized property | ✅ | ✅ | Triggers menu publish via linked options |
| 6 | Customized option | ✅ | ✅ | Triggers menu publish for affected menus |
| 7 | Compound price changes | ✅ | ✅ | Triggers menu publish for affected menus |
| 8 | Restaurant contract | ❌ | ✅ | **OMISSION**: Not documented but exists in code |

**Additional Finding:**
- The document omits the 8th event type: `restaurant_contract` (cdc_event_handler.go:121-122)
- This event handles restaurant contract activation/deactivation and distribution mode changes
- Handler: `handleRestaurantContractChanged()` at lines 497-515

---

### ✅ 2. Batching Strategy

**Document Claims:**
- Time Window: 10-second batching
- Job ID Format: `menu_{menuID}_rest_{restaurantID}_t_{timestamp}`
- Deduplication: Uses `asynq.ErrTaskIDConflict`
- Retry Policy: 3 retries with 30-second timeout

**Verification Status:** ✅ **100% VERIFIED**

**Source:** `internal/job/publish_menu_changed.go`

```go
// Line 53: 10-second time window
executedAt := nextTimeWindowStart(time.Now(), 10*time.Second)

// Line 54: Job ID format
jobId := fmt.Sprintf("menu_%d_rest_%d_t_%d", menuID, restaurantID, executedAt.Unix())

// Line 19: 3 retries
Retry: pkg.PtrOf(3)

// Line 21: 30-second timeout
Timeout: 30 * time.Second
```

**Deduplication verified in** `internal/consumer/cdc_event_handler.go:557-559`:
```go
if err = m.queue.EnqueueAt(publishJob, executedAt); err != nil {
    if errors.Is(err, asynq.ErrTaskIDConflict) {
        return nil  // Skip duplicate jobs
    }
    // ...
}
```

---

### ✅ 3. Menu Publishing Payload Structure

**Document Example:**
```json
{
  "channel_id": "deliverect_channel_123",
  "channel_name": "DoorDash",
  "publish_menus": [
    {
      "menu_id": 456,
      "menu_name": "Lunch Menu",
      "menu_foreign_name": "午餐菜单"
    }
  ],
  "operator_id": "user_789",
  "operator_name": "John Doe",
  "operator_email": "john@example.com"
}
```

**Verification Status:** ✅ **VERIFIED**

**Source:** `internal/extsvc/harbor/thirdparty_client.go:77-84`

```go
type PublishMenuInput struct {
    ChannelId     string            `json:"channel_id"`
    ChannelName   string            `json:"channel_name"`
    PublishMenus  []PublishMenuItem `json:"publish_menus"`
    OperatorId    string            `json:"operator_id"`
    OperatorName  string            `json:"operator_name"`
    OperatorEmail string            `json:"operator_email"`
}

type PublishMenuItem struct {
    MenuId          uint32 `json:"menu_id"`
    MenuName        string `json:"menu_name"`
    MenuForeignName string `json:"menu_foreign_name"`
}
```

**Perfect match** between documentation and actual implementation.

---

### ✅ 4. API Endpoints

**Document Claims:** 8 API endpoints for Deliverect integration

**Verification Status:** ✅ **100% VERIFIED**

**Source:** `internal/extsvc/harbor/deliverect.go`

| # | Endpoint | Method | Purpose | Code Line | Status |
|---|----------|--------|---------|-----------|--------|
| 1 | `/api/internal/v1/deliverect/restaurants/{id}/channel_links` | GET | Get delivery channels | 57 | ✅ |
| 2 | `/api/internal/v1/deliverect/restaurants/{id}/channel_links/busymode` | PUT | Toggle store open/closed | 74 | ✅ |
| 3 | `/api/internal/v1/deliverect/restaurants/{id}/opening_hours` | GET | Get business hours | 88 | ✅ |
| 4 | `/api/internal/v1/deliverect/restaurants/{id}/opening_hours` | PUT | Update business hours | 108 | ✅ |
| 5 | `/api/internal/v1/deliverect/restaurants/{id}/holidays` | GET | Get blackout days | 122 | ✅ |
| 6 | `/api/internal/v1/deliverect/restaurants/{id}/holidays` | PUT | Update blackout days | 138 | ✅ |
| 7 | `/api/internal/v1/deliverect/restaurants/{id}/publish_menus` | POST | Publish menu to channel | 152 | ✅ |
| 8 | `/api/internal/v1/deliverect/restaurants/{id}/setting` | GET | Get restaurant settings | 166 | ✅ |

**Authentication:** All write operations use `WithAuthAdminHeader("catalog")` (Lines 75, 109, 139, 153)

---

### ✅ 5. Data Structures

**Document Claims:** Complex nested structures for channels, menus, hours, and holidays

**Verification Status:** ✅ **VERIFIED**

**Sources:**
- `internal/extsvc/harbor/thirdparty_client.go`
- `internal/model/third_party_*.go`

#### Channel Structure
```go
// harbor/thirdparty_client.go:40-54
type Channel struct {
    ID     string        `json:"id"`
    Name   string        `json:"name"`
    Active bool          `json:"active"`      // Busy mode flag
    Status string        `json:"status"`
    Menus  []ChannelMenu `json:"menus"`
}

type ChannelMenu struct {
    MenuId          uint32 `json:"menu_id"`
    MenuName        string `json:"menu_name"`
    MenuForeignName string `json:"menu_foreign_name"`  // Multi-language support
}
```

#### Business Hours Structure
```go
// harbor/thirdparty_client.go:56-63
type OpeningHour struct {
    DayOfWeek string `json:"day_of_week"`
    StartTime string `json:"start_time"`  // Hour in a day, e.g., "08:00"
    EndTime   string `json:"end_time"`
    Active    bool   `json:"active"`      // Can enable/disable per day
}
```

#### Holiday/Blackout Structure
```go
// harbor/thirdparty_client.go:65-70
type Holiday struct {
    StartTime time.Time `json:"start_time"`
    EndTime   time.Time `json:"end_time"`
}
```

**Key Finding:** All structures match documentation. The code includes helpful comments about field semantics.

---

### ✅ 6. Business Hours Management Features

**Document Claims:**
- Full day-of-week granularity
- Active/inactive flags per day
- Start/end times per day

**Verification Status:** ✅ **VERIFIED**

**Source:** `internal/service/third_party_setting.go:137-146`

```go
// Process openingHours data
for _, openingHourItem := range openingHours {
    // start time may be bigger than end time meaning it's a next day time, which is allowed
    resp.BusinessHours = append(resp.BusinessHours, model.ThirdPartyBusinessHour{
        Acitve:    openingHourItem.Active,  // Per-day active flag
        StartAt:   openingHourItem.StartTime,
        EndAt:     openingHourItem.EndTime,
        DayOfWeek: openingHourItem.DayOfWeek,  // Full day-of-week granularity
    })
}
```

**Additional Finding:** Code handles edge case where start time > end time (overnight hours)

---

### ✅ 7. Holiday/Blackout Management

**Document Claims:**
- Get/Update holiday periods
- Explicit Deliverect feature

**Verification Status:** ✅ **VERIFIED**

**Source:** `internal/service/third_party_setting.go:148-158`

```go
// Process holidays data
for _, holidayItem := range holidays {
    if holidayItem.StartTime.After(holidayItem.EndTime) {
        // in case of start time > end time, skip it
        continue
    }
    resp.BlackoutDays = append(resp.BlackoutDays, model.ThirdPartyBlackoutDay{
        StartAt: holidayItem.StartTime,
        EndAt:   holidayItem.EndTime,
    })
}
```

**Additional Finding:** Code validates that StartTime ≤ EndTime before adding blackout days

---

### ✅ 8. Audit Trail with Operator Tracking

**Document Claims:**
- Comprehensive operator tracking (ID, name, email) for compliance

**Verification Status:** ✅ **VERIFIED**

**Source:** `internal/extsvc/harbor/thirdparty_client.go:77-84`

```go
type PublishMenuInput struct {
    ChannelId     string            `json:"channel_id"`
    ChannelName   string            `json:"channel_name"`
    PublishMenus  []PublishMenuItem `json:"publish_menus"`
    OperatorId    string            `json:"operator_id"`     // ✅ Audit trail
    OperatorName  string            `json:"operator_name"`   // ✅ Audit trail
    OperatorEmail string            `json:"operator_email"`  // ✅ Audit trail
}
```

**Additional Finding:** User activity logging in `third_party_setting.go:189-204`
```go
if err := svc.loggingSvcCli.PublishThirdPartyPublishCatalogUserActivity(ctx, loggingExtSvc.ThirdPartyPublishCatalogUserActivity{
    RestaurantID: restaurantID,
    BusinessType: catalogIdlv1.BusinessType_BUSINESS_TYPE_THIRD_PARTY,
    ActivityType: activityType,
    SourceID:     channelItem.ID,
    SourceType:   catalogIdlv1.SourceType_SOURCE_TYPE_THIRD_PARTY_CHANNEL,
    SourceName:   channelItem.Name,
    Notes:        req.RequestSource.String(),
}); err != nil {
    // logger error here, not a critical error, so we don't return error
    logger.Error(ctx, "", "publish_third_party_setting_catalog_user_activity_error_in_update_third_party_setting", ...)
}
```

---

### ✅ 9. Concurrent Data Fetching with Timeout Protection

**Document Claims:**
- Graceful error handling
- Concurrent data fetching with timeout protection

**Verification Status:** ✅ **VERIFIED**

**Source:** `internal/service/third_party_setting.go:65-116`

```go
func (svc *ThirdPartySettingSvcImpl) GetThirdPartySetting(ctx context.Context, restaurantID string) (resp model.ThirdPartySetting, err error) {
    // Create context with timeout
    ctxWithTimeout, cancel := context.WithTimeout(ctx, thirdPartyAPITimeout)  // 10 seconds
    defer cancel()

    // Create errgroup with timeout context
    g, ctx := errgroup.WithContext(ctxWithTimeout)

    // Concurrently fetch channels data
    var channels harborExtSvc.Channels
    g.Go(func() error {
        ch, err := svc.thirdPartyCli.GetChannelsByRestaurantID(ctx, restaurantID)
        // ... error handling
    })

    // Concurrently fetch openingHours data
    var openingHours harborExtSvc.OpeningHours
    g.Go(func() error {
        oh, err := svc.thirdPartyCli.GetOpeningHoursByRestaurantID(ctx, restaurantID)
        // ... error handling
    })

    // Concurrently fetch holidays data
    var holidays harborExtSvc.Holidays
    g.Go(func() error {
        hd, err := svc.thirdPartyCli.GetHolidaysByRestaurantID(ctx, restaurantID)
        // ... error handling
    })

    // Wait for all goroutines to complete or return error
    if err = g.Wait(); err != nil {
        // Log specific message if timeout error occurs
        if ctxWithTimeout.Err() == context.DeadlineExceeded {
            logger.Error(ctx, "", "ThirdPartySettingSvcImpl.GetThirdPartySetting", "Request timeout", err)
        }
        return
    }
    // ...
}
```

**Key Findings:**
- Uses `errgroup.WithContext` for concurrent fetching (lines 71-107)
- 10-second timeout protection (line 67, constant at line 19)
- Explicit timeout detection and logging (lines 112-114)
- Fetches 3 resources in parallel (channels, opening hours, holidays)

---

### ✅ 10. Multi-Language Support

**Document Claims:**
- Menu names support foreign names (e.g., Chinese)

**Verification Status:** ✅ **VERIFIED**

**Source:** Multiple locations

```go
// harbor/thirdparty_client.go:48-52
type ChannelMenu struct {
    MenuId          uint32 `json:"menu_id"`
    MenuName        string `json:"menu_name"`
    MenuForeignName string `json:"menu_foreign_name"`  // ✅ Multi-language
}

// model/third_party_channel.go:12-16
type ThirdPartyChannelMenu struct {
    MenuId          uint32 `json:"menu_id"`
    MenuName        string `json:"menu_name"`
    MenuForeignName string `json:"menu_foreign_name"`  // ✅ Multi-language
}
```

---

## Issues and Omissions

### ⚠️ Minor Issues

1. **Missing Restaurant Contract Event**
   - **Location:** CDC event handler documentation
   - **Impact:** Low - The 8th CDC event type is not mentioned
   - **Recommendation:** Add `restaurant_contract` to the event type list
   - **Code Reference:** `cdc_event_handler.go:121-122`

2. **Typo in Model Field Name**
   - **Location:** `internal/model/third_party_business_hour.go:7`
   - **Issue:** Field is named `Acitve` instead of `Active` (typo)
   - **Impact:** None (JSON tag is correct, only Go field name is misspelled)
   - **Recommendation:** Fix typo if backward compatibility allows

### ℹ️ Undocumented Features

1. **Empty Holidays Array Behavior**
   - **Location:** `harbor/deliverect.go:137-149`
   - **Finding:** Sending empty `holidays` array deletes all holidays
   - **Code Comment:** Line 26 states "if holidays is empty, then delete all holidays"
   - **Recommendation:** Document this behavior in the comparison

2. **Opening Hours Edge Case**
   - **Location:** `third_party_setting.go:139`
   - **Finding:** System allows `start_time > end_time` for overnight hours
   - **Code Comment:** "start time may be bigger than end time meaning it's a next day time, which is allowed"
   - **Recommendation:** Add to documentation

3. **Non-Critical Error Handling**
   - **Location:** `third_party_setting.go:198-204`
   - **Finding:** User activity logging failures are logged but don't fail the operation
   - **Code Comment:** "not a critical error, so we don't return error"
   - **Recommendation:** Document error handling philosophy

---

## Architecture Insights

### 1. Event-Driven Menu Synchronization

The system uses a sophisticated CDC-based approach:

```
Database Change → CDC Event → Kafka → Event Handler → Job Queue (with batching) → Harbor API → Deliverect
```

**Key Benefits:**
- Real-time propagation (with 10s batching)
- Durability (persisted jobs survive restarts)
- Deduplication (prevents duplicate menu publishes)
- Automatic retry (3 attempts with 30s timeout)

### 2. Job ID Design Pattern

```go
jobId := fmt.Sprintf("menu_%d_rest_%d_t_%d", menuID, restaurantID, executedAt.Unix())
```

This creates time-bucketed job IDs that:
- Ensure uniqueness per menu per restaurant per time window
- Enable automatic deduplication via task ID conflict
- Allow temporal analysis of job execution patterns

### 3. Dependency Graph Complexity

The CDC handler manages a complex dependency graph:

```
Menu
├── Menu Categories
│   └── Categories
│       └── Meal Instances
│           ├── Customized Properties
│           │   └── Customized Options
│           └── Compound Prices
└── Restaurant Contract
```

Changes at any level trigger recalculation of dependent entities.

---

## Recommendations

### 1. Documentation Updates

Add the following to the comparison document:

```markdown
### Undocumented CDC Event Type
- **Restaurant Contract Changes**: Triggers when contract activation status or distribution mode changes
  - Handler: `handleRestaurantContractChanged()`
  - Impact: Affects meal availability across contracts
  - Use case: Activating/deactivating delivery platforms

### Edge Cases
- **Overnight Hours**: Business hours support `start_time > end_time` for operations spanning midnight
- **Empty Holidays**: Sending empty array deletes all blackout days (intentional "clear all" behavior)
- **Concurrent Fetches**: All GET operations use 10-second timeout with graceful failure
```

### 2. Code Quality Improvements

```go
// Fix typo in model/third_party_business_hour.go:7
type ThirdPartyBusinessHour struct {
    StartAt   string `json:"start_at"`
    EndAt     string `json:"end_at"`
    DayOfWeek string `json:"day_of_week"`
    Active    bool   `json:"active"`  // Changed from "Acitve"
}
```

### 3. Additional Testing Scenarios

Based on code analysis, ensure tests cover:
- Time-window deduplication edge cases
- Overnight business hours (start > end)
- Empty holiday array deletion
- Timeout handling in concurrent fetches
- Restaurant contract activation/deactivation

---

## Final Verdict

### Accuracy Score: 98/100

**Strengths:**
- ✅ All technical details are accurate
- ✅ Code examples match actual implementation
- ✅ API endpoints correctly documented
- ✅ Data structures accurately represented

**Minor Deductions:**
- ⚠️ Missing restaurant contract CDC event type (-1 point)
- ⚠️ Edge cases not documented (-1 point)

### Confidence Level: **Very High**

All claims have been verified against actual source code. The comparison document is suitable for technical decision-making with the minor additions noted above.

---

## Appendix: File References

| Component | File Path | Lines |
|-----------|-----------|-------|
| CDC Event Handler | `internal/consumer/cdc_event_handler.go` | 66-692 |
| Job Definition | `internal/job/publish_menu_changed.go` | 1-74 |
| Harbor Client | `internal/extsvc/harbor/deliverect.go` | 1-181 |
| Third Party Service | `internal/service/third_party_setting.go` | 1-272 |
| Model Definitions | `internal/extsvc/harbor/thirdparty_client.go` | 40-88 |
| Model Definitions | `internal/model/third_party_*.go` | Various |

---

**Generated:** 2026-01-19
**Verification Method:** Static code analysis with cross-referencing
**Codebase Version:** master branch (commit 107d189c)
