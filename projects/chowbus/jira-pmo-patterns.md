# Chowbus Jira PMO Patterns

Quick reference for navigating the Chowbus Jira (chowbus.atlassian.net).

## Project Spaces

| Key | Name | Tickets | Focus | Key People |
|-----|------|:-------:|-------|------------|
| **YAAS** | POS | 2,485+ | Core POS product, payments, promotions, client apps | Nan Sun, Yu Zhang, Yaxuan Han, Simon Xiong |
| **PM** | Project Matters | 1,221+ | Platform infra, dev tools, MDM, SDK, promo engine | Ming Zhou, Yunchen Wang, Chiyu He, Popeye Liu |
| **CHOW** | ChowOps | 133+ | Internal ops workflows, approval flows, ticket system | Yang Li, Deqiang Ze |
| **MAG** | MAGI | 56+ | AI-powered menu processing & intelligence | Yang Liu, Xu Chen, Xin Chen |

**YAAS is the gravitational center** — most product features land here. PM handles cross-cutting platform work. CHOW is internal tooling/ops. MAG is focused AI work.

## Issue Hierarchy

```
Theme (strategic direction, customfield_10586)
  └── Epic (hierarchyLevel: 1)     — OKR-aligned initiative
       └── Story (hierarchyLevel: 0)    — one deployable/testable chunk
            └── Sub-task (hierarchyLevel: -1) — actual work item engineers pick up
```

## Epic Naming Convention

```
[Q{N} OKR {team}] {theme} - {feature} - {client}
```

Examples:
```
[Q1 OKR 平台组] 增收3.0 - 促销活动支持同一时段多个活动 - 奈雪的茶
[Q1 OKR 前台组] All-in-1：员工管理
[Q1 OKR 创新组] AI Menu MVP接入系统
```

Components: quarter number, team name (Chinese), strategic theme, specific feature, enterprise client (if client-driven).

## Teams

| Team | Chinese | Owns |
|------|---------|------|
| Platform Team | 平台组 | Revenue growth, promotions, OpenAPI, reports |
| Frontend Team | 前台组 | C2C product, AI integration, employee mgmt, Chowbus GO |
| Innovation Team | 创新组 | AI Menu, combo meals, allergen labels |

## Themes

Two levels of thematic grouping:

**Custom field** (`customfield_10586`) — strategic dropdown:

| Value | Meaning |
|-------|---------|
| Efficiency and Optimization | Internal process improvement, automation |

**Embedded in epic names** — more granular:

| Theme | Description |
|-------|-------------|
| 增收 3.0 (Revenue Growth 3.0) | Promotions, coupons, loyalty, addon discounts |
| 夯实基础 (Strengthen Foundation) | Reports, OpenAPI, stability |
| 安全/风控/合规 (Security/Risk/Compliance) | 2FA, audit logs, fraud detection |
| 大客需求 (Enterprise Client Requests) | Client-specific features |
| All-in-1 | Employee management, integrated features |
| 落地加拿大 (Canada Expansion) | Multi-tax, Stripe, localization |

Enterprise clients driving the roadmap: 奈雪的茶 (Nayuki), 猛男集团 (Mengnan), 小龙坎 (Xiaolongkan), 茉莉奶白 (Jasmine Milk White), 沪上阿姨 (Aunt Shanghai).

## Creating a Story

| Field | Convention | Example |
|-------|-----------|---------|
| Summary | Feature name, optionally `[scope]` suffix | `限时领券活动[server]` |
| Parent | Epic via `customfield_10014` | `YAAS-60` |
| Sprint | `Sprint{YYYYMMDD}-{YYYYMMDD}` (biweekly) | `Sprint20260303-20260316` |
| Description | Often **null** — detail lives in sub-tasks | — |
| Assignee | Tech lead or primary developer | Set at creation |
| Start Date | `customfield_10015` | `2026-02-26` |
| Due Date | Sprint end date | `2026-03-02` |
| Priority | P3 default, P2 for urgent | P3 |

### Story Naming Patterns

```
# Feature + version
分小费2.0-支持按照gross sales分配小费

# Feature + [scope]
限时领券活动[server]

# Platform tool + version
dev-platform SSO与权限管理-v2

# Integration
Approval Flow External Integration (Fraud Detection System)

# QA/release (prefixed)
【QA】3月9号版本：发版回归验证和发版支持
```

## Creating Sub-tasks

Sub-tasks follow a **phase-based decomposition** pattern. Standard lifecycle:

```
方案调研/熟悉需求  →  方案设计  →  接口设计  →  功能开发  →  自测&联调  →  提测
(Research)         (Design)    (API design)  (Dev)       (Self-test)   (Submit QA)
                                                                        ↓
                                                                    【QA】测试执行
                                                                    (QA Execution)
```

### Sub-task Naming Prefixes

```
# Phase names
技术方案编写            → Architecture/design doc
方案设计/方案调研       → Design/research
接口设计               → API design
功能开发               → Feature development
自测&联调              → Self-test & integration
提测                   → Submit for QA

# QA scope
【QA】测试用例编写      → Test case writing
【QA】测试执行          → Test execution

# Platform/client scope
【POS App】...          → POS App scope
【Tablet】...           → Tablet scope
[Catalog] ...           → Catalog service scope
[Harbor] ...            → Harbor service scope
[QR] ...                → QR ordering scope
```

### Real Example: YAAS-2108 (Tip Split 2.0)

```
1. 分小费2.0 - 技术方案编写              (Design/Arch)
2. 【QA】测试用例编写                     (QA Test Design)
3. 分小费2.0 - 新增分出规则 - 功能开发     (Dev)
4. 【POS App】支持gross sales的流程       (Client Dev)
5. 分小费2.0 - 新增分出规则 - 自测&联调    (Self-test)
6. 【QA】测试执行                         (QA Execution)
```

Time estimates tracked via `aggregatetimeoriginalestimate` (in seconds).

### Quick Template for a Well-Formed Story + Sub-tasks

```
Story:
  Summary:    {feature}-{specific behavior}
  Parent:     YAAS-XXX (Epic)
  Sprint:     Sprint{YYYYMMDD}-{YYYYMMDD}
  Priority:   P3
  Assignee:   {tech lead}
  Start Date: {sprint start}
  Due Date:   {sprint end}

Sub-tasks (in execution order):
  1. {feature} - 技术方案编写
  2. 【QA】{feature} - 测试用例编写
  3. {feature} - 功能开发
  4. 【{client app}】支持{feature}的流程
  5. {feature} - 自测&联调
  6. 【QA】{feature} - 测试执行
```

## Custom Fields

| Field ID | Name | Usage |
|----------|------|-------|
| `customfield_10014` | Epic Link / Parent | Links Story to Epic |
| `customfield_10015` | Start Date | When work begins |
| `customfield_10586` | Theme | Strategic category dropdown |
| `customfield_10620` | Team | Team identifier (e.g., TD) |

## Sprint Naming

```
Sprint{YYYYMMDD}-{YYYYMMDD}
```

Biweekly cadence. Example: `Sprint20260303-20260316`.

## Status Workflow

```
To Do → In Analysis → In Development → Testing → Ready for Test → Ready for Release → Done
```

Most tickets sit in **To Do** — backlog-heavy, pull-based system where teams pick from prioritized backlogs.

## Priority Scale

| Level | Usage |
|-------|-------|
| P0 | Critical / production down |
| P1 | High urgency |
| P2 | Urgent features |
| P3 | **Default** (~95% of tickets) |

Priority is effectively managed through **sprint/OKR assignment**, not ticket priority level. YAAS has a dedicated `[System] Incident` issue type for production issues, separate from planned work.

## Key People

| Person | Space | Role / Focus |
|--------|-------|-------------|
| Nan Sun | YAAS | POS product |
| Yu Zhang | YAAS | POS product |
| Yaxuan Han | YAAS | POS product |
| Simon Xiong | YAAS | POS product |
| Ming Zhou | PM | Platform infra |
| Yunchen Wang | PM | Platform / dev tools |
| Chiyu He | PM | Platform |
| Popeye Liu | PM | Platform |
| Yang Li | CHOW | Internal ops |
| Deqiang Ze | CHOW | Internal ops |
| Yang Liu | MAG | AI menu |
| Xu Chen | MAG | AI menu |
| Xin Chen | MAG | AI menu |

## Cross-Cutting Themes (Q1 2026)

| Theme | Spaces | Description |
|-------|--------|-------------|
| 增收 3.0 (Revenue Growth 3.0) | YAAS, PM | Promo engine refactor, coupons, addon discounts |
| Security / 2FA | YAAS | 2FA for Super Admin, risk controls, audit logging |
| Canada Expansion | YAAS | Multi-tax, Stripe settlement, Kiosk adaptation |
| AI Menu | MAG, YAAS | AI menu digitization, quality checks, multi-image |
| Platform Stability | YAAS, PM | Trading stability, rate limiting, cloud migration |
| Dev Platform | PM, CHOW | SSO/permissions, dashboard components, MDM |
| Deliverect Integration | YAAS | Syncing combos and menus to third-party platforms |
