# GRC 治理风险合规系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 GRC（Governance, Risk, and Compliance）治理风险合规系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
GRC 系统作为企业级应用体系的治理风险合规平台，负责合规管理、风险管理、内控管理、审计管理等合规业务，与 OA、HR、FMS 系统紧密协同，确保企业运营符合法律法规和内部政策要求。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 合规管理 | 合规政策、合规检查、合规培训 | 高 |
| 2 | 风险管理 | 风险识别、风险评估、风险应对 | 高 |
| 3 | 内控管理 | 内控框架、控制活动、缺陷管理 | 高 |
| 4 | 审计管理 | 审计计划、审计执行、审计报告 | 高 |
| 5 | 政策管理 | 政策发布、政策更新、政策归档 | 高 |
| 6 | 合规报表 | 合规统计、风险看板、审计报告 | 中 |
| 7 | 预警管理 | 合规预警、风险预警、异常监控 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 响应时间 < 200ms |
| 可用性 | 99.9% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **事件驱动**: 通过消息队列实现系统间协同

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 合规模块 | 合规管理 | 政策、检查、培训 |
| 风险模块 | 风险管理 | 识别、评估、应对 |
| 内控模块 | 内控管理 | 框架、控制、缺陷 |
| 审计模块 | 审计管理 | 计划、执行、报告 |
| 政策模块 | 政策管理 | 发布、更新、归档 |
| 报表模块 | 合规报表 | 统计、看板 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/grc/
  │   │   │   ├── controller/
  │   │   │   │   ├── ComplianceController.java   # 合规管理
  │   │   │   │   ├── RiskController.java        # 风险管理
  │   │   │   │   ├── ControlController.java     # 内控管理
  │   │   │   │   ├── AuditController.java       # 审计管理
  │   │   │   │   └── PolicyController.java      # 政策管理
  │   │   │   ├── service/
  │   │   │   │   ├── ComplianceService.java
  │   │   │   │   ├── RiskService.java
  │   │   │   │   ├── ControlService.java
  │   │   │   │   ├── AuditService.java
  │   │   │   │   └── PolicyService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── GrcApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── risk/                            # 风险组件
  │   │   │   └── RiskMatrix.vue
  │   ├── views/
  │   │   ├── compliance/                      # 合规管理
  │   │   │   └── index.vue
  │   │   ├── risk/                            # 风险管理
  │   │   │   └── index.vue
  │   │   ├── control/                         # 内控管理
  │   │   │   └── index.vue
  │   │   ├── audit/                           # 审计管理
  │   │   │   └── index.vue
  │   │   ├── policy/                          # 政策管理
  │   │   │   └── index.vue
  │   │   └── dashboard/                       # 合规看板
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ComplianceService (合规服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createComplianceCheck` | 创建合规检查 | `ComplianceCheckCreateRequest request` | `ComplianceCheckResponse` | 抛出`BusinessException` |
| `executeCheck` | 执行检查 | `Long checkId` | `ComplianceCheckResponse` | 抛出`CheckNotFoundException` |
| `getComplianceStatus` | 获取合规状态 | `ComplianceStatusRequest request` | `ComplianceStatusResponse` | - |

#### 5.1.2 RiskService (风险服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `identifyRisk` | 识别风险 | `RiskIdentifyRequest request` | `RiskResponse` | 抛出`BusinessException` |
| `assessRisk` | 评估风险 | `Long riskId, RiskAssessmentRequest request` | `RiskResponse` | 抛出`RiskNotFoundException` |
| `mitigateRisk` | 风险应对 | `Long riskId, RiskMitigationRequest request` | `RiskResponse` | 抛出`RiskNotFoundException` |
| `getRiskMatrix` | 获取风险矩阵 | `RiskMatrixRequest request` | `RiskMatrixResponse` | - |

#### 5.1.3 AuditService (审计服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createAuditPlan` | 创建审计计划 | `AuditPlanCreateRequest request` | `AuditPlanResponse` | 抛出`BusinessException` |
| `executeAudit` | 执行审计 | `Long planId` | `AuditExecutionResponse` | 抛出`AuditPlanNotFoundException` |
| `generateAuditReport` | 生成审计报告 | `Long planId` | `AuditReportResponse` | 抛出`AuditPlanNotFoundException` |

### 5.2 DTO 结构定义

**ComplianceCheckCreateRequest（创建合规检查请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| checkName | String | 检查名称 | 非空 |
| complianceArea | String | 合规领域 | 非空 |
| checkType | String | 检查类型 | 非空 |
| frequency | String | 检查频率 | 非空 |
| scope | String | 检查范围 | 非空 |
| criteria | List<CheckCriteriaRequest> | 检查标准 | 非空 |

**RiskIdentifyRequest（风险识别请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| riskName | String | 风险名称 | 非空 |
| riskCategory | String | 风险类别 | 非空 |
| riskDescription | String | 风险描述 | 非空 |
| businessArea | String | 业务领域 | 非空 |
| impact | String | 影响程度 | 非空 |
| probability | String | 发生概率 | 非空 |

**AuditPlanCreateRequest（创建审计计划请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| planName | String | 计划名称 | 非空 |
| auditType | String | 审计类型 | 非空 |
| auditScope | String | 审计范围 | 非空 |
| plannedStartDate | LocalDateTime | 计划开始时间 | 非空 |
| plannedEndDate | LocalDateTime | 计划结束时间 | 非空 |
| auditorIds | List<Long> | 审计人员 ID 列表 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 合规检查表 (grc_compliance_check)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 检查 ID |
| check_name | VARCHAR(200) | NOT NULL | 检查名称 |
| compliance_area | VARCHAR(50) | NOT NULL | 合规领域 |
| check_type | VARCHAR(50) | NOT NULL | 检查类型 |
| frequency | VARCHAR(20) | NOT NULL | 检查频率 |
| scope | VARCHAR(500) | NOT NULL | 检查范围 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| last_executed_at | DATETIME | - | 上次执行时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 风险登记表 (grc_risk_register)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 风险 ID |
| risk_name | VARCHAR(200) | NOT NULL | 风险名称 |
| risk_category | VARCHAR(50) | NOT NULL | 风险类别 |
| risk_description | TEXT | NOT NULL | 风险描述 |
| business_area | VARCHAR(50) | NOT NULL | 业务领域 |
| impact | VARCHAR(20) | NOT NULL | 影响程度 |
| probability | VARCHAR(20) | NOT NULL | 发生概率 |
| risk_level | VARCHAR(20) | NOT NULL | 风险等级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| mitigated_at | DATETIME | - | 应对时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 内控控制表 (grc_internal_control)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 控制 ID |
| control_name | VARCHAR(200) | NOT NULL | 控制名称 |
| control_type | VARCHAR(50) | NOT NULL | 控制类型 |
| control_description | TEXT | NOT NULL | 控制描述 |
| risk_id | BIGINT | FOREIGN KEY | 关联风险 ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| effectiveness | VARCHAR(20) | - | 有效性 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 审计计划表 (grc_audit_plan)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 计划 ID |
| plan_name | VARCHAR(200) | NOT NULL | 计划名称 |
| audit_type | VARCHAR(50) | NOT NULL | 审计类型 |
| audit_scope | VARCHAR(500) | NOT NULL | 审计范围 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| planned_start_date | DATETIME | NOT NULL | 计划开始时间 |
| planned_end_date | DATETIME | NOT NULL | 计划结束时间 |
| actual_start_date | DATETIME | - | 实际开始时间 |
| actual_end_date | DATETIME | - | 实际结束时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 政策表 (grc_policy)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 政策 ID |
| policy_no | VARCHAR(50) | UNIQUE, NOT NULL | 政策编号 |
| policy_name | VARCHAR(200) | NOT NULL | 政策名称 |
| policy_category | VARCHAR(50) | NOT NULL | 政策类别 |
| policy_content | TEXT | NOT NULL | 政策内容 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| effective_date | DATE | NOT NULL | 生效日期 |
| expiry_date | DATE | - | 到期日期 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 合规管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/grc/compliance/checks` | GET | ComplianceController.java | 查询合规检查列表 |
| `/api/grc/compliance/checks` | POST | ComplianceController.java | 创建合规检查 |
| `/api/grc/compliance/checks/{id}/execute` | POST | ComplianceController.java | 执行检查 |
| `/api/grc/compliance/status` | GET | ComplianceController.java | 获取合规状态 |

### 7.2 风险管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/grc/risk/register` | GET | RiskController.java | 查询风险登记列表 |
| `/api/grc/risk/register` | POST | RiskController.java | 识别风险 |
| `/api/grc/risk/{id}/assess` | POST | RiskController.java | 评估风险 |
| `/api/grc/risk/{id}/mitigate` | POST | RiskController.java | 风险应对 |
| `/api/grc/risk/matrix` | GET | RiskController.java | 获取风险矩阵 |

### 7.3 内控管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/grc/control/internal` | GET | ControlController.java | 查询内控控制列表 |
| `/api/grc/control/internal` | POST | ControlController.java | 创建内控控制 |

### 7.4 审计管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/grc/audit/plans` | GET | AuditController.java | 查询审计计划列表 |
| `/api/grc/audit/plans` | POST | AuditController.java | 创建审计计划 |
| `/api/grc/audit/plans/{id}/execute` | POST | AuditController.java | 执行审计 |
| `/api/grc/audit/plans/{id}/report` | GET | AuditController.java | 生成审计报告 |

### 7.5 政策管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/grc/policy` | GET | PolicyController.java | 查询政策列表 |
| `/api/grc/policy` | POST | PolicyController.java | 创建政策 |
| `/api/grc/policy/{id}` | PUT | PolicyController.java | 更新政策 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 合规管理 | compliance:read, compliance:write | 查看和创建合规检查 |
| 风险管理 | risk:read, risk:write | 查看和管理风险 |
| 审计管理 | audit:read, audit:write | 查看和管理审计 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| OA | REST API | 审批流程 |
| HR | REST API | 合规培训记录 |
| FMS | REST API | 合规费用管理 |

---

**文档结束**