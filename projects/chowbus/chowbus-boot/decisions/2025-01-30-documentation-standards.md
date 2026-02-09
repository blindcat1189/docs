# Documentation Standards for chowbus-boot

## Date
2025-01-30

## Status
Accepted

## Context

chowbus-boot is a middleware library with 10 modules. Current documentation has several issues:
- Language inconsistency (root README in English, module READMEs in Chinese)
- No centralized changelog
- No migration guides for breaking changes
- Some modules lack documentation entirely

## Decision

### 1. Documentation Structure

```
chowbus-boot/
├── README.md                    # Root overview (bilingual)
├── CHANGELOG.md                 # Release-level changelog
├── docs/
│   ├── migrations/
│   │   └── v0.x-to-v1.x.md     # Version migration guides
│   └── decisions/               # ADRs (existing)
├── common/
│   └── chowbus-boot-base/
│       ├── README.md            # Module doc (bilingual)
│       └── CHANGELOG.md         # Module changelog
├── auth/
│   ├── chowbus-boot-biz-auth/
│   │   ├── README.md
│   │   └── CHANGELOG.md
│   ├── chowbus-boot-console-auth/
│   │   ├── README.md
│   │   └── CHANGELOG.md
│   └── chowbus-boot-dubbo-auth/
│       ├── README.md
│       └── CHANGELOG.md
├── middleware/
│   ├── chowbus-boot-cache/
│   │   ├── README.md
│   │   └── CHANGELOG.md
│   ├── chowbus-boot-config/
│   │   ├── README.md
│   │   └── CHANGELOG.md
│   ├── chowbus-boot-db/
│   │   ├── README.md
│   │   └── CHANGELOG.md
│   ├── chowbus-boot-event/
│   │   ├── README.md
│   │   └── CHANGELOG.md
│   └── chowbus-boot-exception/
│       ├── README.md
│       └── CHANGELOG.md
└── ai/
    └── chowbus-boot-mcp/
        ├── README.md
        └── CHANGELOG.md
```

### 2. Bilingual Format

Side-by-side format with English paragraph followed by Chinese summary:

```markdown
## Quick Start / 快速开始

Add the dependency to your `pom.xml`:

在 `pom.xml` 中添加依赖：

\`\`\`xml
<dependency>
    <groupId>com.chowbus</groupId>
    <artifactId>chowbus-boot-cache</artifactId>
    <version>${chowbus-boot.version}</version>
</dependency>
\`\`\`
```

**Conventions:**
- Section headers: `## English / 中文`
- Prose: English paragraph, then Chinese summary (not word-for-word translation)
- Code blocks: Shared (no duplication)
- Tables: English only (Chinese column headers if needed)
- Comments in code: English only

### 3. Changelog Format

**Root CHANGELOG.md** - Release summary:

```markdown
# Changelog

All notable changes to chowbus-boot.
chowbus-boot 的所有重要变更记录。

## [1.0.0] - 2025-01-30

### Breaking Changes / 破坏性变更
- `ChowbusError.type` renamed to `code` ([Migration Guide](docs/migrations/v0.x-to-v1.x.md))

### Added / 新增
- Response-level `meta` field in Result models

### Changed / 变更
- JSON output uses `code` instead of `type` for errors

### Fixed / 修复
- None

### Modules Updated / 模块更新
- chowbus-boot-base: Result model improvements
- chowbus-boot-exception: Updated to use new error field naming
```

**Module CHANGELOG.md** - Detailed module changes:

```markdown
# chowbus-boot-cache Changelog

## [1.0.0] - 2025-01-30

### Added / 新增
- Feature description

### Changed / 变更
- Change description

### Configuration Changes / 配置变更
| Old | New | Notes |
|-----|-----|-------|
| `old.property` | `new.property` | Description |
```

### 4. Migration Guide Format

Located in `docs/migrations/v{from}-to-v{to}.md`:

```markdown
# Migration Guide: v0.x to v1.0.0
# 迁移指南：v0.x 到 v1.0.0

## Overview / 概述

Brief description of what changed.

## Breaking Changes / 破坏性变更

### 1. Change Title

**Before / 之前:**
\`\`\`java
// old code
\`\`\`

**After / 之后:**
\`\`\`java
// new code
\`\`\`

**Migration steps / 迁移步骤:**
1. Step one
2. Step two

**Compatibility / 兼容性:**
- Backward compatibility notes

## Configuration Changes / 配置变更

| Module | Old Property | New Property | Notes |
|--------|--------------|--------------|-------|

## Deprecations / 废弃项

| Item | Deprecated In | Remove In | Replacement |
|------|---------------|-----------|-------------|
```

### 5. Module README Template

```markdown
# Module Name

Brief description (1-2 sentences)
简介 (Chinese summary)

## Features / 功能特性
- Feature list with checkmarks

## Quick Start / 快速开始
- Maven dependency
- Minimal configuration
- Basic usage example

## Configuration / 配置说明
- Table of all properties with defaults

## Usage Examples / 使用示例
- Common use cases with code samples

## Architecture / 架构说明 (optional)
- How it works internally (for complex modules)

## Best Practices / 最佳实践
- Dos and don'ts

## Troubleshooting / 常见问题 (optional)
- Common issues and solutions
```

## Implementation Plan

**Priority order:**

1. Create `docs/migrations/v0.x-to-v1.x.md` (capture current breaking changes)
2. Create root `CHANGELOG.md`
3. Update root `README.md` with bilingual format
4. High priority modules: base, exception (no README currently)
5. Convert existing Chinese READMEs to bilingual (biz-auth, console-auth, cache, config, db, event)
6. Create new READMEs for missing modules (dubbo-auth, mcp)
7. Add per-module `CHANGELOG.md` files

**Module status:**

| Module | Current State | Priority |
|--------|---------------|----------|
| chowbus-boot-base | No README | High |
| chowbus-boot-exception | No README | High |
| chowbus-boot-biz-auth | Chinese README | Medium |
| chowbus-boot-console-auth | Chinese README | Medium |
| chowbus-boot-dubbo-auth | No README | Medium |
| chowbus-boot-cache | Chinese README | Medium |
| chowbus-boot-config | Chinese README | Medium |
| chowbus-boot-db | Chinese README | Medium |
| chowbus-boot-event | Chinese README | Medium |
| chowbus-boot-mcp | No README | Low |

## Consequences

### Positive
- Consistent documentation across all modules
- Clear upgrade path for consumers with migration guides
- Changelog provides visibility into changes
- Bilingual format serves both English and Chinese-speaking developers

### Negative
- Higher maintenance burden (two languages)
- Need to update changelog with each release

### Mitigations
- Chinese summaries are brief (not full translations)
- Changelog updates can be part of PR workflow
