# Monitor 系统：架构图

**更新日期**：2026-01-18

---

## 系统架构 (Mermaid)

```mermaid
flowchart TB
    subgraph UpstreamPhase1["第一期上游系统 (日志来源)"]
        subgraph BackendServices["Backend Services"]
            subgraph Service1["Catalog Service"]
                Nginx1["Nginx"]
                App1["Application"]
                FB1["Fluent Bit"]
            end
            subgraph Service2["POS Service"]
                Nginx2["Nginx"]
                App2["Application"]
                FB2["Fluent Bit"]
            end
            BackendNote["上报数据：
            POS HTTP 接口日志
            Catalog HTTP 接口日志"]
        end
        subgraph FrontendApps["Frontend Apps"]
            POSApp["POS App"]
            BrandedApp["Branded App"]
            FrontendNote["上报数据：
            1. 业务行为埋点日志
            2. HTTP 接口日志"]
        end
    end

    subgraph LogCollection["日志采集层"]
        WebService["Serverless Log Collector"]
    end

    Kafka["MSK (Apache Kafka)"]

    subgraph MonitorService["Monitor Service"]
        subgraph Collector["monitor-collector"]
            Consumer["LogConsumer
            @KafkaListener"]
            Groovy["Groovy Script Engine
            (Parse & Extract)"]
            URLFilter["URL Filter Service
            (Match Business Actions)"]
            Masking["Data Masking
            (Sensitive Fields)"]
            Loader["StarRocks Stream Load
            (Batch Insert)"]
        end

        subgraph ConfigDB["monitor-service + MySQL"]
            MySQL[("MySQL
            配置数据")]
            Rules["Alert Rules"]
            Actions["Business Actions"]
            MaskRules["Masking Rules"]
        end

        subgraph AlertEngine["monitor-job (告警引擎)"]
            Quartz["Quartz Scheduler"]
            Evaluator["Rule Evaluator
            (MVEL Expression)"]
        end

        subgraph Notification["monitor-notify (通知中心)"]
            Slack["Slack"]
            WeChat["企业微信"]
            Zoom["Zoom"]
        end

        subgraph UI["monitor-ui (前端)"]
            Dashboard["Vue 3 Dashboard
            (Ant Design Vue)"]
        end
    end

    subgraph Storage["StarRocks (日志存储)"]
        LogTable[("business_action_logs")]
    end

    subgraph Downstream["下游系统"]
        NoDownstream["第一期暂无下游系统取数"]
    end

    subgraph Users["内部用户"]
        InternalUser["内部用户"]
    end

    Internet["Public Internet"]

    %% Frontend log flow
    POSApp -->|"Frontend Events"| WebService
    BrandedApp -->|"Frontend Events"| WebService
    WebService --> Kafka

    %% Backend service log flow
    Nginx1 -->|"Access Logs"| FB1
    Nginx2 -->|"Access Logs"| FB2
    FB1 --> Kafka
    FB2 --> Kafka

    %% Monitor consumption flow
    Kafka --> Consumer

    Consumer --> Groovy
    Groovy --> URLFilter
    URLFilter -.->|"Load Config"| Actions
    URLFilter --> Masking
    Masking -.->|"Load Rules"| MaskRules
    Masking --> Loader

    %% Storage
    Loader --> LogTable

    %% Alert flow
    Quartz -->|"Trigger"| Evaluator
    Evaluator -.->|"Load Rules"| Rules
    Evaluator -->|"Query"| LogTable
    Evaluator --> Slack
    Evaluator --> WeChat
    Evaluator --> Zoom

    %% UI flow
    InternalUser -->|"Access"| Internet
    Internet -->|"HTTPS"| Dashboard
    Dashboard -.->|"API"| MySQL
    Dashboard -->|"Query"| LogTable

    %% Downstream (none for phase 1)
    LogTable -.->|"暂无"| NoDownstream

    %% Styling
    classDef upstreamStyle fill:#e1f5fe,stroke:#01579b
    classDef fluentStyle fill:#f3e5f5,stroke:#7b1fa2
    classDef kafkaStyle fill:#e8f5e9,stroke:#2e7d32
    classDef monitorStyle fill:#fce4ec,stroke:#c2185b
    classDef storageStyle fill:#fff8e1,stroke:#f57f17
    classDef configStyle fill:#e3f2fd,stroke:#1565c0
    classDef alertStyle fill:#ffebee,stroke:#c62828
    classDef notifyStyle fill:#f1f8e9,stroke:#558b2f
    classDef uiStyle fill:#ede7f6,stroke:#4527a0
    classDef downstreamStyle fill:#eceff1,stroke:#607d8b
    classDef noteStyle fill:#fffde7,stroke:#fbc02d,stroke-dasharray: 5 5

    class Nginx1,Nginx2,App1,App2,POSApp,BrandedApp upstreamStyle
    class FB1,FB2,WebService fluentStyle
    class Kafka kafkaStyle
    class Consumer,Groovy,URLFilter,Masking,Loader monitorStyle
    class LogTable storageStyle
    class MySQL,Rules,Actions,MaskRules configStyle
    class Quartz,Evaluator alertStyle
    class Slack,WeChat,Zoom notifyStyle
    class Dashboard uiStyle
    class NoDownstream downstreamStyle
    class InternalUser,Internet uiStyle
    class BackendNote,FrontendNote noteStyle
```

---

## 第一期范围说明

### 1.1 上游系统 (日志来源)

| 上游系统 | Kafka Topic | 日志类型 | 说明 |
|----------|-------------|----------|------|
| Catalog Service | monitor.log.catalog | Nginx Access Log | 目录服务HTTP请求日志 |
| POS Service | monitor.log.pos | Nginx Access Log | POS服务HTTP请求日志 |
| POS App / Branded App | monitor.log.pos-app | Frontend Events | 前端埋点事件日志 |

### 1.2 下游系统 (数据消费)

| 下游系统 | 取数方式 | 说明 |
|----------|----------|------|
| **第一期暂无** | - | 当前无其他系统从Monitor取数 |

> 如后续有下游系统需要取数，需评估数据脱敏策略及访问权限。

---

## 存储说明

| 表名 | 说明 |
|------|------|
| business_action_logs | 所有业务日志统一存储 |

---

## 模块说明

| 模块 | 说明 |
|------|------|
| monitor-collector | Kafka消费、日志解析、脱敏、写入StarRocks |
| monitor-service | 核心业务逻辑、配置管理、API接口 |
| monitor-job | Quartz定时任务、告警规则评估 |
| monitor-notify | 统一通知服务 (Slack/企业微信/Zoom) |
| monitor-ui | Vue 3 前端界面 |

## 数据流说明

1. **日志采集**: 上游服务 → Fluent Bit / Serverless Collector → Kafka
2. **日志处理**: Consumer → Groovy解析 → URL匹配 → 数据脱敏 → 批量写入
3. **存储**: 所有日志统一写入 business_action_logs 表
4. **告警触发**: Quartz定时 → 加载规则 → 查询日志 → 评估条件 → 多渠道通知
5. **数据查询**: Dashboard → API/直接查询 → StarRocks
