# Internal API: Operation Log Dictionary Endpoints

# 内部 API：操作日志字典接口

---

## Overview / 概述

The Monitor system exposes two internal dictionary endpoints for audit operation logs:
- **`/dict/modules`** — returns a **3-level tree** of module → category → subcategory, derived from enabled `audit_operation_config` entries with i18n labels resolved from `sys_dict`.
- **`/dict/types`** — returns a flat list of operation types from `sys_dict`.

Monitor 系统提供两个内部字典接口：
- **`/dict/modules`** — 返回基于 `audit_operation_config` 启用配置构建的**三级树**：模块 → 分类 → 子分类，i18n 标签从 `sys_dict` 解析。
- **`/dict/types`** — 返回 `sys_dict` 中的操作类型平铺列表。

**Authentication**: None required — access is controlled at the network/infra level (internal VPC only).

**认证**: 无需认证 — 通过网络层（内部 VPC）控制访问。

---

## Endpoints / 接口

### 1. List Module Category Tree / 获取模块分类树

| Field | Value |
|-------|-------|
| Method | `GET` |
| Path | `/api/internal/v1/operation-logs/dict/modules` |
| Parameters | None |
| Data Source | `audit_operation_config` (enabled only) + `sys_dict` for i18n labels |

Returns a **3-level tree** (module → category → subcategory) built from distinct `(operation_module, first_category, second_category)` triples in enabled `audit_operation_config` entries. Each level is sorted by `sys_dict.sort_order`.

返回基于启用的 `audit_operation_config` 中 `(operation_module, first_category, second_category)` 去重三元组构建的**三级树**。每层按 `sys_dict.sort_order` 排序。

#### Response / 响应

```json
{
  "success": true,
  "data": [
    {
      "name": "data_security",
      "label_zh": "数据安全",
      "label_en": "Data Security",
      "categories": [
        {
          "name": "report",
          "label_zh": "报表",
          "label_en": "Report",
          "subcategories": [
            {
              "name": "employee_report.performance",
              "label_zh": "员工业绩",
              "label_en": "Performance"
            }
          ]
        }
      ]
    }
  ]
}
```

#### ModuleCategoryTreeVO Fields / 字段说明

| Field | Type | Description |
|-------|------|-------------|
| `name` | String | Module raw value (= `operation_module` in config) / 模块原始值 |
| `label_zh` | String | Chinese label from `sys_dict` (fallback to `name`) / 中文标签 |
| `label_en` | String | English label from `sys_dict` (fallback to `name`) / 英文标签 |
| `categories` | List | First-level categories / 一级分类列表 |

#### CategoryVO Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | String | Category raw value (= `first_category` in config) / 分类原始值 |
| `label_zh` | String | Chinese label / 中文标签 |
| `label_en` | String | English label / 英文标签 |
| `subcategories` | List | Second-level subcategories / 二级子分类列表 |

#### SubcategoryVO Fields

| Field | Type | Description |
|-------|------|-------------|
| `name` | String | Subcategory raw value (= `second_category` in config) / 子分类原始值 |
| `label_zh` | String | Chinese label / 中文标签 |
| `label_en` | String | English label / 英文标签 |

### 2. List Types / 获取操作类型列表

| Field | Value |
|-------|-------|
| Method | `GET` |
| Path | `/api/internal/v1/operation-logs/dict/types` |
| Parameters | None |
| Dict Key | `audit_operation_type` |

Returns all **enabled** operation type dictionary entries, sorted by `sort_order`.

返回所有**已启用**的操作类型字典项，按 `sort_order` 排序。

#### Response / 响应

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "dict_key": "audit_operation_type",
      "dict_value": "create",
      "note": "{\"zh\":\"创建\",\"en\":\"Create\"}",
      "note_zh": "创建",
      "note_en": "Create",
      "is_enable": true,
      "sort_order": 1,
      "created_at": "2025-01-15T08:00:00.000+00:00",
      "updated_at": "2025-01-15T08:00:00.000+00:00"
    }
  ]
}
```

> **Note**: chowbus-boot 1.0.0 applies global snake_case JSON serialization via Fastjson2. All field names are automatically converted from camelCase to snake_case.

---

## i18n Label Behavior / 国际化标签逻辑

Labels for the modules tree come from `sys_dict` entries with keys `audit_operation_module`, `audit_first_category`, and `audit_second_category`. The `note` field can be:

1. **JSON format**: `{"zh":"菜单管理","en":"Menu Management"}` — parsed into `label_zh` and `label_en`
2. **Plain text**: `"菜单管理"` — treated as `label_zh`, `label_en` falls back to the raw value

If no dict entry exists for a given value, both `label_zh` and `label_en` fall back to the raw value string.

---

## Usage with Operation Log Query / 与操作日志查询配合使用

The `name` from the tree can be passed as the `module` parameter to the operation log query endpoint:

```
GET /api/internal/v1/operation-logs?module={name}&start_at=...&end_at=...
```

**Example workflow / 使用流程**:

1. Call `/dict/modules` to get the module tree / 调用 `/dict/modules` 获取模块树
2. Display as cascading dropdowns using `label_zh` or `label_en` / 使用 `label_zh` 或 `label_en` 展示为级联下拉框
3. When user selects a module, pass its `name` as the `module` query parameter / 用户选择后，将 `name` 作为 `module` 查询参数

```javascript
// Example: Cascading dropdown population
const res = await fetch('/api/internal/v1/operation-logs/dict/modules');
const { data: modules } = await res.json();

// Top level: modules
const moduleOptions = modules.map(m => ({
  label: locale === 'zh' ? m.label_zh : m.label_en,
  value: m.name,
  children: m.categories.map(c => ({
    label: locale === 'zh' ? c.label_zh : c.label_en,
    value: c.name,
    children: c.subcategories.map(s => ({
      label: locale === 'zh' ? s.label_zh : s.label_en,
      value: s.name,
    })),
  })),
}));
```

---

## Error Handling / 错误处理

On success, `success` is `true` and `data` contains the list (may be empty).

On error, the response follows the standard `SingleResult` error format:

```json
{
  "success": false,
  "data": null,
  "errors": [
    {
      "code": "BAD_REQUEST",
      "title": "start_at and end_at are required"
    }
  ]
}
```

Both endpoints have no required parameters and query only enabled entries, so errors are rare. Possible causes:

- **500**: Database connectivity issues / 数据库连接问题
- **404**: Incorrect path (check URL) / 路径错误

---

## Service Details / 服务信息

| Item | Value |
|------|-------|
| Service | `monitor` |
| Namespace | `menu-ai` |
| Internal base URL (staging) | `http://monitor.menu-ai.svc.cluster.local:8090` |
| Controller | `InternalOperationLogController` |
| Source | `monitor-start/.../controller/internal/InternalOperationLogController.java` |
| Key Service | `AuditOperationConfigService.getModuleCategoryTree()` (for `/dict/modules`) |
| Framework | chowbus-boot 1.0.0 (`SingleResult`, `BizException`, `CommonErrorCode`) |
