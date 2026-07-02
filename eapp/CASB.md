# CASB 云访问安全代理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 CASB（Cloud Access Security Broker）云访问安全代理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
CASB 系统作为企业级应用体系的云安全代理平台，负责云应用访问控制、数据安全、威胁检测和合规管理，与 SSO、GRC 系统紧密协同，确保企业云环境的安全和合规。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 云应用发现 | 自动发现、分类识别、风险评估 | 高 |
| 2 | 访问控制 | 身份认证、授权管理、访问策略 | 高 |
| 3 | 数据安全 | 数据加密、数据分类、数据脱敏 | 高 |
| 4 | 威胁检测 | 异常行为检测、威胁告警、实时监控 | 高 |
| 5 | 合规管理 | 合规检查、审计日志、合规报告 | 高 |
| 6 | 数据治理 | 数据生命周期、数据泄露防护 | 中 |
| 7 | API 安全 | API 监控、API 防护、API 审计 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 响应时间 < 100ms，支持高并发 |
| 可用性 | 99.9% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **代理模式**: 作为云应用访问的安全代理

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 发现模块 | 云应用发现 | 自动发现、分类识别 |
| 访问模块 | 访问控制 | 认证、授权、策略 |
| 数据模块 | 数据安全 | 加密、分类、脱敏 |
| 威胁模块 | 威胁检测 | 异常检测、告警 |
| 合规模块 | 合规管理 | 检查、审计、报告 |
| API模块 | API安全 | 监控、防护、审计 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/casb/
  │   │   │   ├── controller/
  │   │   │   │   ├── DiscoveryController.java   # 云应用发现
  │   │   │   │   ├── AccessController.java      # 访问控制
  │   │   │   │   ├── DataSecurityController.java # 数据安全
  │   │   │   │   ├── ThreatController.java      # 威胁检测
  │   │   │   │   └── ComplianceController.java  # 合规管理
  │   │   │   ├── service/
  │   │   │   │   ├── DiscoveryService.java
  │   │   │   │   ├── AccessControlService.java
  │   │   │   │   ├── DataSecurityService.java
  │   │   │   │   ├── ThreatDetectionService.java
  │   │   │   │   └── ComplianceService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── CasbApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── dashboard/                        # 安全看板组件
  │   │   │   └── SecurityDashboard.vue
  │   ├── views/
  │   │   ├── discovery/                       # 云应用发现
  │   │   │   └── index.vue
  │   │   ├── access/                          # 访问控制
  │   │   │   └── index.vue
  │   │   ├── datasecurity/                    # 数据安全
  │   │   │   └── index.vue
  │   │   ├── threat/                          # 威胁检测
  │   │   │   └── index.vue
  │   │   └── compliance/                      # 合规管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 DiscoveryService (发现服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `discoverCloudApps` | 发现云应用 | `DiscoveryRequest request` | `List<CloudAppResponse>` | 抛出`DiscoveryException` |
| `classifyApp` | 分类云应用 | `String appId, ClassificationRequest request` | `CloudAppResponse` | 抛出`AppNotFoundException` |
| `assessRisk` | 风险评估 | `String appId` | `RiskAssessmentResponse` | 抛出`AppNotFoundException` |
| `getCloudApps` | 查询云应用 | `CloudAppSearchRequest request` | `Page<CloudAppResponse>` | - |

#### 5.1.2 AccessControlService (访问控制服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createAccessPolicy` | 创建访问策略 | `AccessPolicyCreateRequest request` | `AccessPolicyResponse` | 抛出`BusinessException` |
| `applyPolicy` | 应用策略 | `Long policyId` | `AccessPolicyResponse` | 抛出`PolicyNotFoundException` |
| `evaluateAccess` | 评估访问 | `AccessEvaluationRequest request` | `AccessDecisionResponse` | - |

#### 5.1.3 ThreatDetectionService (威胁检测服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `detectAnomaly` | 检测异常行为 | `BehaviorRequest request` | `AnomalyDetectionResponse` | - |
| `getAlerts` | 获取威胁告警 | `AlertSearchRequest request` | `Page<AlertResponse>` | - |
| `acknowledgeAlert` | 确认告警 | `Long alertId` | `AlertResponse` | 抛出`AlertNotFoundException` |

### 5.2 DTO 结构定义

**DiscoveryRequest（发现请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| scanType | String | 扫描类型(FULL/INCREMENTAL) | 非空 |
| timeout | Integer | 超时时间(秒) | 可选 |

**CloudAppResponse（云应用响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| id | String | 应用 ID |
| name | String | 应用名称 |
| type | String | 应用类型 |
| provider | String | 服务提供商 |
| riskLevel | String | 风险等级 |
| discoveredAt | LocalDateTime | 发现时间 |
| lastAccessTime | LocalDateTime | 最后访问时间 |

**AccessPolicyCreateRequest（创建访问策略请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| policyName | String | 策略名称 | 非空 |
| policyType | String | 策略类型 | 非空 |
| appIds | List<String> | 应用 ID 列表 | 非空 |
| conditions | List<PolicyCondition> | 策略条件 | 非空 |
| actions | List<PolicyAction> | 执行动作 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 云应用表 (casb_cloud_app)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | VARCHAR(50) | PRIMARY KEY | 应用 ID |
| name | VARCHAR(200) | NOT NULL | 应用名称 |
| type | VARCHAR(50) | NOT NULL | 应用类型 |
| provider | VARCHAR(100) | NOT NULL | 服务提供商 |
| description | VARCHAR(500) | - | 描述 |
| risk_level | VARCHAR(20) | NOT NULL | 风险等级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| discovered_at | DATETIME | NOT NULL | 发现时间 |
| last_access_time | DATETIME | - | 最后访问时间 |

#### 6.1.2 访问策略表 (casb_access_policy)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 策略 ID |
| policy_name | VARCHAR(100) | NOT NULL | 策略名称 |
| policy_type | VARCHAR(50) | NOT NULL | 策略类型 |
| conditions | TEXT | NOT NULL | 策略条件(JSON) |
| actions | TEXT | NOT NULL | 执行动作(JSON) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 威胁告警表 (casb_threat_alert)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 告警 ID |
| alert_type | VARCHAR(50) | NOT NULL | 告警类型 |
| severity | VARCHAR(20) | NOT NULL | 严重程度 |
| message | VARCHAR(500) | NOT NULL | 告警消息 |
| app_id | VARCHAR(50) | NOT NULL | 关联应用 ID |
| user_id | BIGINT | - | 关联用户 ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| detected_at | DATETIME | NOT NULL | 检测时间 |
| acknowledged_at | DATETIME | - | 确认时间 |

#### 6.1.4 审计日志表 (casb_audit_log)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 日志 ID |
| event_type | VARCHAR(50) | NOT NULL | 事件类型 |
| resource | VARCHAR(200) | NOT NULL | 资源 |
| action | VARCHAR(50) | NOT NULL | 操作 |
| user_id | BIGINT | NOT NULL | 用户 ID |
| app_id | VARCHAR(50) | - | 应用 ID |
| ip_address | VARCHAR(50) | - | IP 地址 |
| result | VARCHAR(20) | NOT NULL | 结果 |
| occurred_at | DATETIME | NOT NULL | 发生时间 |

---

## 7. API 接口设计

### 7.1 云应用发现接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/casb/discovery/apps` | GET | DiscoveryController.java | 查询云应用列表 |
| `/api/casb/discovery/scan` | POST | DiscoveryController.java | 执行应用发现 |
| `/api/casb/discovery/{appId}/risk` | GET | DiscoveryController.java | 获取风险评估 |

### 7.2 访问控制接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/casb/access/policies` | GET | AccessController.java | 查询访问策略 |
| `/api/casb/access/policies` | POST | AccessController.java | 创建访问策略 |
| `/api/casb/access/evaluate` | POST | AccessController.java | 评估访问请求 |

### 7.3 威胁检测接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/casb/threat/alerts` | GET | ThreatController.java | 获取威胁告警 |
| `/api/casb/threat/detect` | POST | ThreatController.java | 检测异常行为 |
| `/api/casb/threat/alerts/{id}/acknowledge` | POST | ThreatController.java | 确认告警 |

### 7.4 合规管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/casb/compliance/check` | POST | ComplianceController.java | 合规检查 |
| `/api/casb/compliance/report` | GET | ComplianceController.java | 获取合规报告 |
| `/api/casb/compliance/logs` | GET | ComplianceController.java | 查询审计日志 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 发现管理 | discovery:read, discovery:write | 查看和管理发现 |
| 访问控制 | access:read, access:write | 查看和管理策略 |
| 威胁检测 | threat:read, threat:write | 查看和处理告警 |

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

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| GRC | REST API | 合规报告同步 |
| ITSM | REST API | 安全告警工单 |

---

**文档结束**