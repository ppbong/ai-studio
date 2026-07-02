# ITSM IT服务管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 ITSM（IT Service Management）IT服务管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
ITSM 系统作为企业级应用体系的 IT 服务管理平台，负责 IT 服务台、事件管理、问题管理、变更管理、知识库等 IT 服务管理业务，与 EAM、OA 系统紧密协同，实现 IT 服务的标准化和规范化管理。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 服务台 | 工单受理、工单分派、工单跟踪 | 高 |
| 2 | 事件管理 | 事件记录、事件分类、事件解决 | 高 |
| 3 | 问题管理 | 问题识别、问题分析、问题解决 | 高 |
| 4 | 变更管理 | 变更申请、变更审批、变更实施 | 高 |
| 5 | 资产管理 | IT 资产、配置项、资产关联 | 高 |
| 6 | 知识库 | 知识文章、知识搜索、知识分享 | 中 |
| 7 | 服务目录 | 服务目录、服务级别、服务报告 | 中 |

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
| 服务台模块 | 服务台 | 工单受理、分派 |
| 事件模块 | 事件管理 | 事件记录、解决 |
| 问题模块 | 问题管理 | 问题识别、分析 |
| 变更模块 | 变更管理 | 变更申请、审批 |
| 资产模块 | 资产管理 | IT 资产、配置项 |
| 知识库模块 | 知识库 | 知识文章、搜索 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/itsm/
  │   │   │   ├── controller/
  │   │   │   │   ├── ServiceDeskController.java # 服务台
  │   │   │   │   ├── IncidentController.java    # 事件管理
  │   │   │   │   ├── ProblemController.java     # 问题管理
  │   │   │   │   ├── ChangeController.java      # 变更管理
  │   │   │   │   ├── AssetController.java       # 资产管理
  │   │   │   │   └── KnowledgeController.java   # 知识库
  │   │   │   ├── service/
  │   │   │   │   ├── ServiceDeskService.java
  │   │   │   │   ├── IncidentService.java
  │   │   │   │   ├── ProblemService.java
  │   │   │   │   ├── ChangeService.java
  │   │   │   │   ├── AssetService.java
  │   │   │   │   └── KnowledgeService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── ItsmApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── ticket/                           # 工单组件
  │   │   │   └── TicketCard.vue
  │   ├── views/
  │   │   ├── servicedesk/                      # 服务台
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── incident/                        # 事件管理
  │   │   │   └── index.vue
  │   │   ├── problem/                         # 问题管理
  │   │   │   └── index.vue
  │   │   ├── change/                          # 变更管理
  │   │   │   └── index.vue
  │   │   ├── asset/                           # 资产管理
  │   │   │   └── index.vue
  │   │   └── knowledge/                       # 知识库
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ServiceDeskService (服务台服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createTicket` | 创建工单 | `TicketCreateRequest request` | `TicketResponse` | 抛出`BusinessException` |
| `assignTicket` | 分派工单 | `Long ticketId, AssignRequest request` | `TicketResponse` | 抛出`TicketNotFoundException` |
| `resolveTicket` | 解决工单 | `Long ticketId, ResolveRequest request` | `TicketResponse` | 抛出`TicketNotFoundException` |
| `getTickets` | 查询工单列表 | `TicketSearchRequest request` | `Page<TicketResponse>` | - |

#### 5.1.2 IncidentService (事件服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createIncident` | 创建事件 | `IncidentCreateRequest request` | `IncidentResponse` | 抛出`BusinessException` |
| `updateIncident` | 更新事件 | `Long incidentId, IncidentUpdateRequest request` | `IncidentResponse` | 抛出`IncidentNotFoundException` |
| `closeIncident` | 关闭事件 | `Long incidentId` | `IncidentResponse` | 抛出`IncidentNotFoundException` |

#### 5.1.3 ChangeService (变更服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createChange` | 创建变更 | `ChangeCreateRequest request` | `ChangeResponse` | 抛出`BusinessException` |
| `submitForApproval` | 提交审批 | `Long changeId` | `ChangeResponse` | 抛出`ChangeNotFoundException` |
| `implementChange` | 实施变更 | `Long changeId, ImplementRequest request` | `ChangeResponse` | 抛出`ChangeNotFoundException` |

### 5.2 DTO 结构定义

**TicketCreateRequest（创建工单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| title | String | 工单标题 | 非空 |
| description | String | 工单描述 | 非空 |
| category | String | 工单分类 | 非空 |
| priority | String | 优先级 | 非空 |
| requesterId | Long | 请求人 ID | 非空 |
| requesterPhone | String | 联系电话 | 可选 |
| assetId | Long | 关联资产 ID | 可选 |

**IncidentResponse（事件响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| id | Long | 事件 ID |
| incidentNo | String | 事件编号 |
| title | String | 事件标题 |
| category | String | 事件分类 |
| priority | String | 优先级 |
| status | String | 状态 |
| requesterId | Long | 请求人 ID |
| assignedTo | Long | 分配给 |
| resolvedAt | LocalDateTime | 解决时间 |
| createdAt | LocalDateTime | 创建时间 |

**ChangeCreateRequest（创建变更请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| title | String | 变更标题 | 非空 |
| description | String | 变更描述 | 非空 |
| changeType | String | 变更类型 | 非空 |
| priority | String | 优先级 | 非空 |
| plannedStartDate | LocalDateTime | 计划开始时间 | 非空 |
| plannedEndDate | LocalDateTime | 计划结束时间 | 非空 |
| requesterId | Long | 请求人 ID | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 工单表 (itsm_ticket)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 工单 ID |
| ticket_no | VARCHAR(50) | UNIQUE, NOT NULL | 工单编号 |
| title | VARCHAR(200) | NOT NULL | 工单标题 |
| description | TEXT | NOT NULL | 工单描述 |
| category | VARCHAR(50) | NOT NULL | 工单分类 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| requester_id | BIGINT | NOT NULL | 请求人 ID |
| assigned_to | BIGINT | - | 分配给 |
| asset_id | BIGINT | - | 关联资产 ID |
| resolved_at | DATETIME | - | 解决时间 |
| closed_at | DATETIME | - | 关闭时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 事件表 (itsm_incident)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 事件 ID |
| incident_no | VARCHAR(50) | UNIQUE, NOT NULL | 事件编号 |
| title | VARCHAR(200) | NOT NULL | 事件标题 |
| description | TEXT | NOT NULL | 事件描述 |
| category | VARCHAR(50) | NOT NULL | 事件分类 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| requester_id | BIGINT | NOT NULL | 请求人 ID |
| assigned_to | BIGINT | - | 分配给 |
| resolution | TEXT | - | 解决方案 |
| resolved_at | DATETIME | - | 解决时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 问题表 (itsm_problem)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 问题 ID |
| problem_no | VARCHAR(50) | UNIQUE, NOT NULL | 问题编号 |
| title | VARCHAR(200) | NOT NULL | 问题标题 |
| description | TEXT | NOT NULL | 问题描述 |
| category | VARCHAR(50) | NOT NULL | 问题分类 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| root_cause | TEXT | - | 根本原因 |
| workaround | TEXT | - | 临时解决方案 |
| permanent_solution | TEXT | - | 永久解决方案 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 变更表 (itsm_change)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 变更 ID |
| change_no | VARCHAR(50) | UNIQUE, NOT NULL | 变更编号 |
| title | VARCHAR(200) | NOT NULL | 变更标题 |
| description | TEXT | NOT NULL | 变更描述 |
| change_type | VARCHAR(50) | NOT NULL | 变更类型 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| requester_id | BIGINT | NOT NULL | 请求人 ID |
| planned_start_date | DATETIME | NOT NULL | 计划开始时间 |
| planned_end_date | DATETIME | NOT NULL | 计划结束时间 |
| actual_start_date | DATETIME | - | 实际开始时间 |
| actual_end_date | DATETIME | - | 实际结束时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 IT 资产表 (itsm_asset)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 资产 ID |
| asset_no | VARCHAR(50) | UNIQUE, NOT NULL | 资产编号 |
| asset_name | VARCHAR(200) | NOT NULL | 资产名称 |
| asset_type | VARCHAR(50) | NOT NULL | 资产类型 |
| model | VARCHAR(100) | - | 型号 |
| serial_number | VARCHAR(100) | - | 序列号 |
| location | VARCHAR(200) | - | 位置 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| owner_id | BIGINT | - | 使用人 ID |
| purchase_date | DATE | - | 采购日期 |
| warranty_expiry | DATE | - | 保修到期日 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 服务台接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/itsm/tickets` | GET | ServiceDeskController.java | 查询工单列表 |
| `/api/itsm/tickets/{id}` | GET | ServiceDeskController.java | 查询工单详情 |
| `/api/itsm/tickets` | POST | ServiceDeskController.java | 创建工单 |
| `/api/itsm/tickets/{id}/assign` | POST | ServiceDeskController.java | 分派工单 |
| `/api/itsm/tickets/{id}/resolve` | POST | ServiceDeskController.java | 解决工单 |

### 7.2 事件管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/itsm/incidents` | GET | IncidentController.java | 查询事件列表 |
| `/api/itsm/incidents/{id}` | GET | IncidentController.java | 查询事件详情 |
| `/api/itsm/incidents` | POST | IncidentController.java | 创建事件 |
| `/api/itsm/incidents/{id}` | PUT | IncidentController.java | 更新事件 |
| `/api/itsm/incidents/{id}/close` | POST | IncidentController.java | 关闭事件 |

### 7.3 变更管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/itsm/changes` | GET | ChangeController.java | 查询变更列表 |
| `/api/itsm/changes/{id}` | GET | ChangeController.java | 查询变更详情 |
| `/api/itsm/changes` | POST | ChangeController.java | 创建变更 |
| `/api/itsm/changes/{id}/approve` | POST | ChangeController.java | 提交审批 |
| `/api/itsm/changes/{id}/implement` | POST | ChangeController.java | 实施变更 |

### 7.4 资产管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/itsm/assets` | GET | AssetController.java | 查询资产列表 |
| `/api/itsm/assets/{id}` | GET | AssetController.java | 查询资产详情 |
| `/api/itsm/assets` | POST | AssetController.java | 创建资产 |

### 7.5 知识库接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/itsm/knowledge/articles` | GET | KnowledgeController.java | 查询知识文章 |
| `/api/itsm/knowledge/articles` | POST | KnowledgeController.java | 创建知识文章 |
| `/api/itsm/knowledge/search` | GET | KnowledgeController.java | 知识搜索 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 工单管理 | ticket:read, ticket:write | 查看和创建工单 |
| 事件管理 | incident:read, incident:write | 查看和管理事件 |
| 变更管理 | change:read, change:write | 查看和管理变更 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| Elasticsearch | 8.x | 全文检索 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| EAM | REST API | IT 资产同步 |
| OA | REST API | 变更审批流程 |
| BI | REST API | IT 服务报表 |

---

**文档结束**