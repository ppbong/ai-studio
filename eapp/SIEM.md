# SIEM 安全信息与事件管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 SIEM（Security Information and Event Management）安全信息与事件管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
SIEM 系统作为企业级应用体系的安全监控平台，负责日志收集、安全事件分析、威胁检测、合规报告等安全管理业务，与 CASB、ITSM、GRC 系统紧密协同，实现企业安全态势的实时监控和威胁响应。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 日志收集 | 多源日志采集、日志标准化、日志存储 | 高 |
| 2 | 事件分析 | 实时分析、关联分析、威胁检测 | 高 |
| 3 | 威胁情报 | 威胁情报整合、威胁匹配、告警升级 | 高 |
| 4 | 安全监控 | 安全态势、实时告警、仪表盘 | 高 |
| 5 | 合规报告 | 合规审计、报表生成、合规验证 | 高 |
| 6 | 响应处置 | 事件响应、工单创建、自动响应 | 中 |
| 7 | 用户行为分析 | UEBA、异常检测、用户画像 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 支持每秒百万级日志处理，延迟 < 500ms |
| 可用性 | 99.9% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **事件驱动**: 通过消息队列实现实时日志处理

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 采集模块 | 日志收集 | 多源采集、标准化 |
| 分析模块 | 事件分析 | 实时分析、关联分析 |
| 情报模块 | 威胁情报 | 情报整合、匹配 |
| 监控模块 | 安全监控 | 态势、告警、仪表盘 |
| 合规模块 | 合规报告 | 审计、报表 |
| 响应模块 | 响应处置 | 事件响应、工单 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/siem/
  │   │   │   ├── controller/
  │   │   │   │   ├── LogController.java        # 日志管理
  │   │   │   │   ├── EventController.java      # 事件管理
  │   │   │   │   ├── AlertController.java      # 告警管理
  │   │   │   │   ├── MonitorController.java    # 安全监控
  │   │   │   │   └── ReportController.java     # 合规报告
  │   │   │   ├── service/
  │   │   │   │   ├── LogCollectionService.java
  │   │   │   │   ├── EventAnalysisService.java
  │   │   │   │   ├── ThreatIntelligenceService.java
  │   │   │   │   ├── AlertService.java
  │   │   │   │   └── ReportService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── SiemApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── timeline/                        # 时间线组件
  │   │   │   └── SecurityTimeline.vue
  │   ├── views/
  │   │   ├── dashboard/                       # 安全仪表盘
  │   │   │   └── index.vue
  │   │   ├── events/                          # 事件管理
  │   │   │   └── index.vue
  │   │   ├── alerts/                          # 告警管理
  │   │   │   └── index.vue
  │   │   ├── logs/                            # 日志查询
  │   │   │   └── index.vue
  │   │   └── reports/                         # 合规报告
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 LogCollectionService (日志收集服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `collectLog` | 收集日志 | `LogCollectRequest request` | `LogResponse` | 抛出`CollectException` |
| `batchCollect` | 批量收集日志 | `List<LogCollectRequest> requests` | `List<LogResponse>` | - |
| `searchLogs` | 查询日志 | `LogSearchRequest request` | `Page<LogResponse>` | - |

#### 5.1.2 EventAnalysisService (事件分析服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `analyzeEvent` | 分析事件 | `EventAnalyzeRequest request` | `EventAnalysisResponse` | - |
| `correlateEvents` | 关联分析 | `CorrelationRequest request` | `CorrelationResponse` | - |
| `detectThreat` | 威胁检测 | `ThreatDetectRequest request` | `ThreatDetectionResponse` | - |

#### 5.1.3 AlertService (告警服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createAlert` | 创建告警 | `AlertCreateRequest request` | `AlertResponse` | 抛出`BusinessException` |
| `getAlerts` | 获取告警列表 | `AlertSearchRequest request` | `Page<AlertResponse>` | - |
| `acknowledgeAlert` | 确认告警 | `Long alertId` | `AlertResponse` | 抛出`AlertNotFoundException` |
| `escalateAlert` | 升级告警 | `Long alertId` | `AlertResponse` | 抛出`AlertNotFoundException` |

### 5.2 DTO 结构定义

**LogCollectRequest（日志收集请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| sourceType | String | 来源类型 | 非空 |
| sourceName | String | 来源名称 | 非空 |
| timestamp | LocalDateTime | 时间戳 | 非空 |
| level | String | 日志级别 | 非空 |
| message | String | 日志消息 | 非空 |
| fields | Map<String, Object> | 自定义字段 | 可选 |

**EventAnalyzeRequest（事件分析请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| eventId | String | 事件 ID | 非空 |
| eventType | String | 事件类型 | 非空 |
| severity | String | 严重程度 | 非空 |
| source | String | 事件来源 | 非空 |
| details | Map<String, Object> | 事件详情 | 可选 |

**AlertCreateRequest（创建告警请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| alertType | String | 告警类型 | 非空 |
| severity | String | 严重程度 | 非空 |
| message | String | 告警消息 | 非空 |
| source | String | 告警来源 | 非空 |
| eventId | String | 关联事件 ID | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 日志表 (siem_log)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 日志 ID |
| source_type | VARCHAR(50) | NOT NULL | 来源类型 |
| source_name | VARCHAR(100) | NOT NULL | 来源名称 |
| timestamp | DATETIME | NOT NULL | 时间戳 |
| level | VARCHAR(20) | NOT NULL | 日志级别 |
| message | TEXT | NOT NULL | 日志消息 |
| fields | TEXT | - | 自定义字段(JSON) |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 事件表 (siem_event)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | VARCHAR(50) | PRIMARY KEY | 事件 ID |
| event_type | VARCHAR(50) | NOT NULL | 事件类型 |
| severity | VARCHAR(20) | NOT NULL | 严重程度 |
| source | VARCHAR(200) | NOT NULL | 事件来源 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| details | TEXT | NOT NULL | 事件详情(JSON) |
| detected_at | DATETIME | NOT NULL | 检测时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 告警表 (siem_alert)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 告警 ID |
| alert_type | VARCHAR(50) | NOT NULL | 告警类型 |
| severity | VARCHAR(20) | NOT NULL | 严重程度 |
| message | VARCHAR(500) | NOT NULL | 告警消息 |
| source | VARCHAR(200) | NOT NULL | 告警来源 |
| event_id | VARCHAR(50) | - | 关联事件 ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| acknowledged_at | DATETIME | - | 确认时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 威胁情报表 (siem_threat_intelligence)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 情报 ID |
| indicator_type | VARCHAR(50) | NOT NULL | 指标类型 |
| indicator_value | VARCHAR(200) | NOT NULL | 指标值 |
| threat_type | VARCHAR(50) | NOT NULL | 威胁类型 |
| confidence | DECIMAL(5,2) | NOT NULL | 置信度 |
| description | VARCHAR(500) | - | 描述 |
| source | VARCHAR(100) | NOT NULL | 情报来源 |
| expires_at | DATETIME | - | 过期时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 日志管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/siem/logs` | GET | LogController.java | 查询日志列表 |
| `/api/siem/logs/collect` | POST | LogController.java | 收集日志 |
| `/api/siem/logs/batch` | POST | LogController.java | 批量收集日志 |

### 7.2 事件管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/siem/events` | GET | EventController.java | 查询事件列表 |
| `/api/siem/events/{id}` | GET | EventController.java | 查询事件详情 |
| `/api/siem/events/analyze` | POST | EventController.java | 分析事件 |
| `/api/siem/events/correlate` | POST | EventController.java | 关联分析 |

### 7.3 告警管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/siem/alerts` | GET | AlertController.java | 获取告警列表 |
| `/api/siem/alerts/{id}` | GET | AlertController.java | 获取告警详情 |
| `/api/siem/alerts/{id}/acknowledge` | POST | AlertController.java | 确认告警 |
| `/api/siem/alerts/{id}/escalate` | POST | AlertController.java | 升级告警 |

### 7.4 合规报告接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/siem/reports/security` | GET | ReportController.java | 安全报告 |
| `/api/siem/reports/compliance` | GET | ReportController.java | 合规报告 |
| `/api/siem/reports/threat` | GET | ReportController.java | 威胁报告 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 日志管理 | log:read, log:write | 查看和收集日志 |
| 事件管理 | event:read, event:write | 查看和分析事件 |
| 告警管理 | alert:read, alert:write | 查看和处理告警 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| Kafka | 3.5+ | 消息队列 |
| Elasticsearch | 8.x | 日志检索 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| CASB | REST API + 消息队列 | 安全事件同步 |
| ITSM | REST API | 告警工单创建 |
| GRC | REST API | 合规报告同步 |

---

**文档结束**