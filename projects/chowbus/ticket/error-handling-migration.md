# Error Handling Migration Guide

This guide helps team members migrate the ticket project from numeric error codes to the ChowBus standard dotted-string error pattern using `chowbus-boot-exception` module (release/1.0.0).

## Table of Contents
- [Current vs Target State](#current-vs-target-state)
- [ChowBus Boot Exception Module Overview](#chowbus-boot-exception-module-overview)
- [Migration Steps](#migration-steps)
- [Code Examples](#code-examples)
- [i18n Support](#i18n-support)
- [Best Practices](#best-practices)

---

## Current vs Target State

### Current State (ticket project)

```java
// Current: Numeric error codes with inline messages
return SingleResult.buildFailure("5000", "表单不存在");
return SingleResult.buildFailure("6001", "未授权访问");
```

**Problems:**
- No type safety - error codes are magic strings
- Messages scattered throughout codebase
- No i18n support
- Inconsistent with company-wide error handling

### Target State (chowbus-boot-exception 1.0.0)

```java
// Target: Type-safe enum + BizException (auto-handled by AOP)
throw new BizException(TicketErrorCode.FORM_NOT_FOUND);
throw new BizException(TicketErrorCode.AUTH_UNAUTHORIZED);
```

**Response format (ChowbusError):**
```json
{
  "success": false,
  "error": {
    "type": "Form.NotFound",
    "title": "表单未找到",
    "detail": "Form not found"
  }
}
```

**Benefits:**
- **No GlobalExceptionHandler needed** - AOP auto-intercepts all Result-returning methods
- Type-safe error codes via enums
- Built-in i18n support (error code = translation key)
- Consistent with company standards

---

## ChowBus Boot Exception Module Overview

### Architecture (release/1.0.0)

```
chowbus-boot-exception/
├── aop/
│   ├── ExceptionHandlingAspect.java  # Auto-intercepts Result-returning methods
│   └── ResultWrapper.java            # Builds failure Result from exceptions
├── code/
│   ├── ErrorCode.java                # Interface: getCode(), getMessage(), getHttpStatus()
│   └── CommonErrorCode.java          # Predefined common error codes
├── exception/
│   ├── BizException.java             # Base business exception
│   ├── ValidationException.java      # Field-level validation errors
│   ├── ResourceNotFoundException.java
│   ├── AuthenticationException.java
│   └── RateLimitException.java
├── i18n/
│   ├── LocaleContext.java            # ThreadLocal for request locale
│   ├── MessageResolver.java          # Interface for message translation
│   └── PropertyMessageResolver.java  # Properties file-based translation
├── filter/
│   └── LocaleFilter.java             # Extracts locale from Accept-Language header
└── translator/
    ├── ExceptionTranslator.java      # SPI for translating framework exceptions
    ├── DataAccessExceptionTranslator.java
    └── DubboExceptionTranslator.java
```

### Key Feature: Auto-Interception

**No annotations required!** The AOP aspect automatically intercepts all methods returning `Result`, `SingleResult`, or `MultiResult`:

```java
// Pointcut definition in ExceptionHandlingAspect.java
@Pointcut("execution(public com.chowbus.boot.base.data.Result+ *(..))")
public void resultReturningMethod() {}
```

Any exception thrown in these methods is automatically caught and converted to a failure Result.

### ErrorCode Interface

```java
public interface ErrorCode {
    String getCode()
     // e.g., "Form.NotFound"
    String getMessage()
  // Default message (debug)
    int getHttpStatus()
  // HTTP status code
}
```

### CommonErrorCode (预定义的通用错误码)

| Enum | Code | HTTP | Use Case |
|------|------|------|----------|
| `INVALID_PARAMETER` | `Request.Param.Invalid` | 400 | Invalid parameter |
| `MISSING_PARAMETER` | `Request.Param.Missing` | 400 | Missing required parameter |
| `VALIDATION_ERROR` | `Validation.Error` | 400 | Validation failed |
| `TOKEN_INVALID` | `Auth.Token.Invalid` | 401 | Invalid token |
| `TOKEN_EXPIRED` | `Auth.Token.Expired` | 401 | Token expired |
| `ACCESS_DENIED` | `Auth.AccessDenied` | 403 | Access denied |
| `RESOURCE_NOT_FOUND` | `Resource.NotFound` | 404 | Resource not found |
| `DUPLICATE_RESOURCE` | `Resource.Duplicate` | 409 | Resource already exists |
| `INTERNAL_ERROR` | `System.Internal` | 500 | Internal server error |

### BizException Class

```java
// Using ErrorCode enum
throw new BizException(CommonErrorCode.RESOURCE_NOT_FOUND);

// With message arguments (for parameterized i18n messages)
throw new BizException(CommonErrorCode.RESOURCE_NOT_FOUND, "Form", formId);

// With custom code and message
throw new BizException("Form.NotFound", "Form does not exist");

// With additional data
throw new BizException("Form.NotFound", "Form does not exist", formData);

// Static factory for internal errors
throw BizException.internal("Unexpected error during form creation");
```

### ChowbusError Response Structure

```java
@Data
@Builder
public class ChowbusError {
    @Deprecated  // Use 'type' instead
    private String code;

    private String type
   // Error code (e.g., "Form.NotFound") - also i18n key
    private String title
  // User-facing translated message
    private String detail
 // Debug message (not user-facing)
    private Map<String, String> meta
 // Additional info (e.g., field errors)
}
```

---

## Migration Steps

### Step 1: Add Dependency

Add `chowbus-boot-exception` to `ticket-service/pom.xml`:

```xml
<dependency>
    <groupId>com.chowbus</groupId>
    <artifactId>chowbus-boot-exception</artifactId>
    <version>1.0.0</version>
</dependency>
```

### Step 2: Enable Module (Optional - enabled by default)

In `application.properties`:

```properties
# Exception module (enabled by default)
chowbus.exception.enabled=true
chowbus.exception.show-internal-errors=false
chowbus.exception.biz-exception-log-level=WARN
chowbus.exception.system-exception-log-level=ERROR

# Locale filter (enabled by default for web apps)
chowbus.exception.locale-filter.enabled=true
```

### Step 3: Create Project-Specific Error Codes

Create `TicketErrorCode.java` in `ticket-client` module:

```java
package com.chowbus.ticket.error;

import com.chowbus.boot.exception.code.ErrorCode;
import lombok.Getter;
import lombok.RequiredArgsConstructor;

/**
 * Ticket system error codes
 *
 * Naming Convention: {Domain}.{SubDomain}.{Specific}
 * - Domain: Flow, Form, Task, Version, Auth
 * - SubDomain: Specific entity or action
 * - Specific: Exact error condition
 */
@Getter
@RequiredArgsConstructor
public enum TicketErrorCode implements ErrorCode {

    // ==================== Flow Errors ====================
    FLOW_NOT_FOUND("Flow.NotFound", "Flow not found", 404),
    FLOW_CODE_EXISTS("Flow.Code.AlreadyExists", "Flow code already exists", 409),
    FLOW_DELETE_HAS_INSTANCES("Flow.Delete.HasInstances", "Cannot delete flow with running instances", 409),
    FLOW_PUBLISH_FAILED("Flow.Publish.Failed", "Failed to publish flow to Flowable engine", 500),

    // ==================== Flow Version Errors ====================
    FLOW_VERSION_NOT_FOUND("Flow.Version.NotFound", "Flow version not found", 404),
    FLOW_VERSION_ALREADY_PUBLISHED("Flow.Version.AlreadyPublished", "Version is already published", 409),
    FLOW_VERSION_NO_DRAFT("Flow.Version.NoDraft", "No draft version found", 404),

    // ==================== Form Errors ====================
    FORM_NOT_FOUND("Form.NotFound", "Form not found", 404),
    FORM_CODE_EXISTS("Form.Code.AlreadyExists", "Form code already exists", 409),
    FORM_CODE_EMPTY("Form.Code.Empty", "Form code cannot be empty", 400),
    FORM_NAME_EMPTY("Form.Name.Empty", "Form name cannot be empty", 400),

    // ==================== Form Version Errors ====================
    FORM_VERSION_NOT_FOUND("Form.Version.NotFound", "Form version not found", 404),
    FORM_VERSION_NO_DRAFT("Form.Version.NoDraft", "No draft version found", 404),

    // ==================== Task Errors ====================
    TASK_NOT_FOUND("Task.NotFound", "Task not found", 404),
    TASK_ALREADY_COMPLETED("Task.AlreadyCompleted", "Task is already completed", 409),

    // ==================== Auth Errors ====================
    AUTH_UNAUTHORIZED("Auth.Unauthorized", "Unauthorized access", 401),
    AUTH_FORBIDDEN("Auth.Forbidden", "Forbidden", 403);

    private final String code;
    private final String message;
    private final int httpStatus;
}
```

### Step 4: Add i18n Translation Files

Create `src/main/resources/i18n/error_messages.properties` (English):

```properties
# Flow Errors
Flow.NotFound=Flow not found
Flow.Code.AlreadyExists=Flow code already exists
Flow.Delete.HasInstances=Cannot delete flow with running instances
Flow.Publish.Failed=Failed to publish flow
Flow.Version.NotFound=Flow version not found
Flow.Version.AlreadyPublished=Version is already published
Flow.Version.NoDraft=No draft version found

# Form Errors
Form.NotFound=Form not found
Form.Code.AlreadyExists=Form code already exists
Form.Code.Empty=Form code cannot be empty
Form.Name.Empty=Form name cannot be empty
Form.Version.NotFound=Form version not found
Form.Version.NoDraft=No draft version found

# Task Errors
Task.NotFound=Task not found
Task.AlreadyCompleted=Task is already completed

# Auth Errors
Auth.Unauthorized=Unauthorized access
Auth.Forbidden=Forbidden
```

Create `src/main/resources/i18n/error_messages_zh_CN.properties` (Chinese):

```properties
# Flow Errors
Flow.NotFound=流程未找到
Flow.Code.AlreadyExists=流程编码已存在
Flow.Delete.HasInstances=无法删除有运行实例的流程
Flow.Publish.Failed=发布流程失败
Flow.Version.NotFound=流程版本未找到
Flow.Version.AlreadyPublished=版本已发布
Flow.Version.NoDraft=没有草稿版本

# Form Errors
Form.NotFound=表单未找到
Form.Code.AlreadyExists=表单编码已存在
Form.Code.Empty=表单编码不能为空
Form.Name.Empty=表单名称不能为空
Form.Version.NotFound=表单版本未找到
Form.Version.NoDraft=没有草稿版本

# Task Errors
Task.NotFound=任务未找到
Task.AlreadyCompleted=任务已完成

# Auth Errors
Auth.Unauthorized=未授权访问
Auth.Forbidden=禁止访问
```

### Step 5: Update Service Implementation

**Before (current code with try-catch and manual Result building):**
```java
@Override
public SingleResult<FormDTO> getFormById(Long id) {
    try {
        FormDO form = this.getById(id);
        if (form == null) {
            return SingleResult.buildFailure("5000", "表单不存在");
        }
        FormDTO dto = ChowbusConvertUtils.convert(form, FormDTO.class);
        return SingleResult.buildSuccess(dto);
    } catch (Exception e) {
        log.error("获取表单详情失败, id: {}", id, e);
        return SingleResult.buildFailure("5000", "获取表单详情失败: " + e.getMessage());
    }
}
```

**After (using BizException - AOP handles everything):**
```java
@Override
public SingleResult<FormDTO> getFormById(Long id) {
    FormDO form = this.getById(id);
    if (form == null) {
        throw new BizException(TicketErrorCode.FORM_NOT_FOUND);
    }

    FormDTO dto = ChowbusConvertUtils.convert(form, FormDTO.class);
    enrichFormDTOWithVersion(dto);
    return SingleResult.buildSuccess(dto);
}
```

**Key changes:**
1. Remove try-catch blocks - AOP handles all exceptions
2. Replace `SingleResult.buildFailure()` with `throw new BizException()`
3. Only return success cases - failures are thrown

### Step 6: Remove GlobalExceptionHandler (if exists)

**Delete any custom GlobalExceptionHandler** - the module provides this automatically via AOP.

---

## Code Examples

### Example 1: Basic Error Throwing

```java
// Simple not found error
if (flow == null) {
    throw new BizException(TicketErrorCode.FLOW_NOT_FOUND);
}

// Validation error
if (StringUtils.isBlank(request.getCode())) {
    throw new BizException(TicketErrorCode.FORM_CODE_EMPTY);
}

// Conflict error
if (existingForm != null) {
    throw new BizException(TicketErrorCode.FORM_CODE_EXISTS);
}
```

### Example 2: With Parameterized Messages

Define message with placeholders in properties:
```properties
Resource.NotFound=Resource not found: {0} with id {1}
```

Use with arguments:
```java
throw new BizException(
    CommonErrorCode.RESOURCE_NOT_FOUND,
    "Form",      // {0}
    formId       // {1}
);
// title: "Resource not found: Form with id 123"
```

### Example 3: Validation with Field Errors

```java
import com.chowbus.boot.exception.exception.ValidationException;

List<ValidationException.FieldError> fieldErrors = new ArrayList<>();
if (StringUtils.isBlank(request.getCode())) {
    fieldErrors.add(new ValidationException.FieldError("code", "Code is required"));
}
if (StringUtils.isBlank(request.getName())) {
    fieldErrors.add(new ValidationException.FieldError("name", "Name is required"));
}

if (!fieldErrors.isEmpty()) {
    throw new ValidationException(fieldErrors);
}
```

Response:
```json
{
  "success": false,
  "error": {
    "type": "Validation.Error",
    "title": "参数验证失败",
    "meta": {
      "field.code": "Code is required",
      "field.name": "Name is required"
    }
  }
}
```

### Example 4: Re-throwing with Context

```java
try {
    flowableService.deploy(flowDefinition);
} catch (FlowableException e) {
    throw new BizException(TicketErrorCode.FLOW_PUBLISH_FAILED, e);
}
```

### Example 5: Complete Service Method

```java
@Override
@Transactional(rollbackFor = Exception.class)
public SingleResult<Long> createForm(FormCreateRequest request) {
    // Validation (throws ValidationException)
    validateFormRequest(request);

    // Check uniqueness
    LambdaQueryWrapper<FormDO> queryWrapper = new LambdaQueryWrapper<>();
    queryWrapper.eq(FormDO::getCode, request.getCode().trim());
    if (this.getOne(queryWrapper) != null) {
        throw new BizException(TicketErrorCode.FORM_CODE_EXISTS);
    }

    // Create form
    FormDO form = ChowbusConvertUtils.convert(request, FormDO.class);
    this.save(form);

    // Create initial version
    createInitialVersion(form.getId(), request.getFormSchema());

    log.info("Created form: id={}, code={}", form.getId(), form.getCode());
    return SingleResult.buildSuccess(form.getId());
}

private void validateFormRequest(FormCreateRequest request) {
    List<ValidationException.FieldError> errors = new ArrayList<>();

    if (StringUtils.isBlank(request.getCode())) {
        errors.add(new ValidationException.FieldError("code", "Form code is required"));
    }
    if (StringUtils.isBlank(request.getName())) {
        errors.add(new ValidationException.FieldError("name", "Form name is required"));
    }

    if (!errors.isEmpty()) {
        throw new ValidationException(errors);
    }
}
```

---

## i18n Support

### How It Works

1. **Request comes in** with `Accept-Language: zh-CN` header
2. **LocaleFilter** extracts locale, stores in `LocaleContext` (ThreadLocal)
3. **Exception thrown** → AOP catches it
4. **MessageResolver** looks up error code in properties file for that locale
5. **Response** contains translated `title` field

### Locale Detection Priority

1. `Accept-Language` header
2. `lang` query parameter (if configured)
3. Default: `Locale.ENGLISH`

### Properties File Location

```
src/main/resources/
└── i18n/
    ├── error_messages.properties         # English (default)
    ├── error_messages_zh_CN.properties   # Chinese Simplified
    ├── error_messages_zh_TW.properties   # Chinese Traditional
    └── error_messages_ja.properties      # Japanese
```

### Fallback Behavior

If no translation found:
1. Try English translation
2. Return error code as-is (e.g., "Form.NotFound")

---

## Best Practices

### 1. Error Code Naming Convention

```
{Domain}.{SubDomain}.{Specific}

Examples:
- Form.NotFound           ✓ Good
- Form.Code.AlreadyExists ✓ Good
- FormNotFoundError       ✗ Bad (not dotted)
- 5000                    ✗ Bad (numeric)
```

### 2. HTTP Status Mapping

| Scenario | HTTP Status | Example Code |
|----------|-------------|--------------|
| Resource not found | 404 | `*.NotFound` |
| Validation failed | 400 | `*.Empty`, `*.Invalid` |
| Already exists | 409 | `*.AlreadyExists` |
| Unauthorized | 401 | `Auth.Unauthorized` |
| Forbidden | 403 | `Auth.Forbidden` |
| Internal error | 500 | `*.Failed`, `System.*` |

### 3. Don't Catch BizException

```java
// ✗ Bad - defeats the purpose of AOP
try {
    validateForm(form);
} catch (BizException e) {
    return SingleResult.buildFailure(e.getErrorCode(), e.getMessage());
}

// ✓ Good - let AOP handle it
validateForm(form)
 // throws BizException if invalid
```

### 4. No Try-Catch for Expected Errors

```java
// ✗ Bad - unnecessary try-catch
try {
    FormDO form = getById(id);
    if (form == null) {
        throw new BizException(TicketErrorCode.FORM_NOT_FOUND);
    }
    return SingleResult.buildSuccess(form);
} catch (BizException e) {
    throw e
 // pointless
} catch (Exception e) {
    throw BizException.internal(e.getMessage(), e);
}

// ✓ Good - clean code, AOP handles exceptions
FormDO form = getById(id);
if (form == null) {
    throw new BizException(TicketErrorCode.FORM_NOT_FOUND);
}
return SingleResult.buildSuccess(form);
```

### 5. Use Specific Exception Types

```java
// For validation errors with field details
throw new ValidationException(fieldErrors);

// For resource not found
throw new ResourceNotFoundException("Form", formId);

// For auth failures
throw new AuthenticationException("Token expired");

// For general business errors
throw new BizException(TicketErrorCode.FLOW_PUBLISH_FAILED);
```

### 6. Logging Strategy

The AOP aspect handles logging automatically:
- `BizException`: Logged at configured level (default: WARN)
- System exceptions: Logged at configured level (default: ERROR)

Configure via properties:
```properties
chowbus.exception.biz-exception-log-level=WARN
chowbus.exception.system-exception-log-level=ERROR
```

---

## Migration Checklist

- [ ] Add `chowbus-boot-exception` dependency (version 1.0.0)
- [ ] Create `TicketErrorCode` enum in `ticket-client`
- [ ] Add i18n properties files (`error_messages.properties`, `error_messages_zh_CN.properties`)
- [ ] Remove custom `GlobalExceptionHandler` (if exists)
- [ ] Migrate `FormServiceImpl` - replace `buildFailure()` with `throw BizException`
- [ ] Migrate `FlowServiceImpl`
- [ ] Migrate `FlowVersionServiceImpl`
- [ ] Migrate `FormVersionServiceImpl`
- [ ] Remove try-catch blocks that just re-wrap exceptions
- [ ] Update unit tests to expect `BizException`
- [ ] Test i18n with `Accept-Language` header

---

## Error Code Mapping Reference

| Old Code | Old Message | New Enum | New Code |
|----------|-------------|----------|----------|
| `5000` | 表单不存在 | `FORM_NOT_FOUND` | `Form.NotFound` |
| `5000` | 表单编码不能为空 | `FORM_CODE_EMPTY` | `Form.Code.Empty` |
| `5000` | 表单名称不能为空 | `FORM_NAME_EMPTY` | `Form.Name.Empty` |
| `5000` | 表单编码已存在 | `FORM_CODE_EXISTS` | `Form.Code.AlreadyExists` |
| `6001` | 未授权访问 | `AUTH_UNAUTHORIZED` | `Auth.Unauthorized` |

---

## FAQ

**Q: Do I need to create a GlobalExceptionHandler?**
A: No! The module provides AOP-based interception automatically. Just throw `BizException` and it will be converted to a proper Result.

**Q: How do I test the i18n?**
A: Send requests with `Accept-Language: zh-CN` header. The `title` field in the error response will be in Chinese.

**Q: What if my method doesn't return Result?**
A: The AOP only intercepts methods returning `Result`, `SingleResult`, or `MultiResult`. For other return types, you'll need manual exception handling.

**Q: Can I add new error codes later?**
A: Yes, just add new enum values to `TicketErrorCode` and corresponding translations in properties files.

---

*Last updated: January 2026 - Based on chowbus-boot release/1.0.0*
