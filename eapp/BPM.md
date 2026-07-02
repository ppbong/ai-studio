# BPM 业务流程管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 BPM（Business Process Management）业务流程管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
BPM 系统作为企业级应用体系的业务流程管理平台，负责流程设计、流程执行、流程监控、流程优化等流程管理业务，与 OA、ERP、CRM 系统紧密协同，实现企业业务流程的自动化和标准化管理。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 流程设计 | 流程建模、表单设计、流程配置 | 高 |
| 2 | 流程执行 | 流程启动、任务分派、流程推进 | 高 |
| 3 | 流程监控 | 实时监控、流程跟踪、异常处理 | 高 |
| 4 | 流程优化 | 流程分析、瓶颈识别、优化建议 | 高 |
| 5 | 流程报表 | 流程统计、效率分析、KPI 报表 | 高 |
| 6 | 流程集成 | 与其他系统集成、数据同步 | 中 |
| 7 | 移动审批 | 移动端审批、离线审批 | 中 |

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
- **事件驱动**: 通过消息队列实现流程事件通知

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 设计模块 | 流程设计 | 流程建模、表单设计 |
| 执行模块 | 流程执行 | 流程启动、任务分派 |
| 监控模块 | 流程监控 | 实时监控、跟踪 |
| 优化模块 | 流程优化 | 分析、瓶颈识别 |
| 报表模块 | 流程报表 | 统计、KPI |
| 集成模块 | 流程集成 | 与其他系统集成 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/bpm/
  │   │   │   ├── controller/
  │   │   │   │   ├── ProcessController.java      # 流程设计
  │   │   │   │   ├── ExecutionController.java    # 流程执行
  │   │   │   │   ├── MonitorController.java      # 流程监控
  │   │   │   │   └── ReportController.java       # 流程报表
  │   │   │   ├── service/
  │   │   │   │   ├── ProcessDesignService.java
  │   │   │   │   ├── ProcessExecutionService.java
  │   │   │   │   ├── ProcessMonitorService.java
  │   │   │   │   └── ProcessReportService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── BpmApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── designer/                        # 流程设计器组件
  │   │   │   └── ProcessDesigner.vue
  │   ├── views/
  │   │   ├── design/                          # 流程设计
  │   │   │   └── index.vue
  │   │   ├── task/                            # 待办任务
  │   │   │   └── index.vue
  │   │   ├── monitor/                         # 流程监控
  │   │   │   └── index.vue
  │   │   ├── report/                          # 流程报表
  │   │   │   └── index.vue
  │   │   └── my/                              # 我的流程
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ProcessDesignService (流程设计服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createProcess` | 创建流程 | `ProcessCreateRequest request` | `ProcessResponse` | 抛出`BusinessException` |
| `deployProcess` | 部署流程 | `Long processId` | `ProcessResponse` | 抛出`ProcessNotFoundException` |
| `updateProcess` | 更新流程 | `Long processId, ProcessUpdateRequest request` | `ProcessResponse` | 抛出`ProcessNotFoundException` |
| `getProcesses` | 查询流程列表 | `ProcessSearchRequest request` | `Page<ProcessResponse>` | - |

#### 5.1.2 ProcessExecutionService (流程执行服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `startProcess` | 启动流程 | `Long processId, ProcessInstanceCreateRequest request` | `ProcessInstanceResponse` | 抛出`ProcessNotFoundException` |
| `completeTask` | 完成任务 | `Long taskId, TaskCompleteRequest request` | `TaskResponse` | 抛出`TaskNotFoundException` |
| `delegateTask` | 委派任务 | `Long taskId, Long delegateUserId` | `TaskResponse` | 抛出`TaskNotFoundException` |
| `getMyTasks` | 获取我的任务 | `Long userId` | `Page<TaskResponse>` | - |

#### 5.1.3 ProcessMonitorService (流程监控服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `getProcessStatus` | 获取流程状态 | `Long instanceId` | `ProcessStatusResponse` | 抛出`InstanceNotFoundException` |
| `getProcessHistory` | 获取流程历史 | `Long instanceId` | `List<ProcessHistoryResponse>` | - |
| `suspendProcess` | 挂起流程 | `Long instanceId` | `ProcessInstanceResponse` | 抛出`InstanceNotFoundException` |
| `terminateProcess` | 终止流程 | `Long instanceId` | `ProcessInstanceResponse` | 抛出`InstanceNotFoundException` |

### 5.2 DTO 结构定义

**ProcessCreateRequest（创建流程请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| processName | String | 流程名称 | 非空 |
| processCode | String | 流程编码 | 非空，唯一 |
| description | String | 流程描述 | 可选 |
| flowchartJson | String | 流程图 JSON | 非空 |
| formJson | String | 表单 JSON | 非空 |

**ProcessInstanceCreateRequest（启动流程请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| processId | Long | 流程 ID | 非空 |
| businessKey | String | 业务主键 | 可选 |
| formData | Map<String, Object> | 表单数据 | 可选 |
| initiatorId | Long | 发起者 ID | 非空 |

**TaskCompleteRequest（完成任务请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| taskId | Long | 任务 ID | 非空 |
| action | String | 动作类型 | 非空 |
| comment | String | 处理意见 | 可选 |
| formData | Map<String, Object> | 表单数据 | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 流程定义表 (bpm_process_definition)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 流程 ID |
| process_name | VARCHAR(200) | NOT NULL | 流程名称 |
| process_code | VARCHAR(50) | UNIQUE, NOT NULL | 流程编码 |
| description | VARCHAR(500) | - | 流程描述 |
| flowchart_json | TEXT | NOT NULL | 流程图 JSON |
| form_json | TEXT | NOT NULL | 表单 JSON |
| status | VARCHAR(20) | NOT NULL | 状态 |
| version | INT | DEFAULT 1 | 版本号 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 流程实例表 (bpm_process_instance)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 实例 ID |
| process_id | BIGINT | FOREIGN KEY, NOT NULL | 流程 ID |
| business_key | VARCHAR(100) | - | 业务主键 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| initiator_id | BIGINT | NOT NULL | 发起者 ID |
| started_at | DATETIME | NOT NULL | 启动时间 |
| completed_at | DATETIME | - | 完成时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 任务表 (bpm_task)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 任务 ID |
| instance_id | BIGINT | FOREIGN KEY, NOT NULL | 实例 ID |
| task_name | VARCHAR(100) | NOT NULL | 任务名称 |
| task_key | VARCHAR(50) | NOT NULL | 任务标识 |
| assignee_id | BIGINT | - | 处理人 ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| due_date | DATETIME | - | 截止日期 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| form_data | TEXT | - | 表单数据(JSON) |
| created_at | DATETIME | NOT NULL | 创建时间 |
| completed_at | DATETIME | - | 完成时间 |

#### 6.1.4 流程历史表 (bpm_process_history)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 历史 ID |
| instance_id | BIGINT | NOT NULL | 实例 ID |
| task_id | BIGINT | - | 任务 ID |
| activity_name | VARCHAR(100) | NOT NULL | 活动名称 |
| activity_type | VARCHAR(50) | NOT NULL | 活动类型 |
| assignee_id | BIGINT | - | 处理人 ID |
| action | VARCHAR(50) | NOT NULL | 动作 |
| comment | VARCHAR(500) | - | 处理意见 |
| occurred_at | DATETIME | NOT NULL | 发生时间 |

---

## 7. API 接口设计

### 7.1 流程设计接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/bpm/processes` | GET | ProcessController.java | 查询流程列表 |
| `/api/bpm/processes/{id}` | GET | ProcessController.java | 查询流程详情 |
| `/api/bpm/processes` | POST | ProcessController.java | 创建流程 |
| `/api/bpm/processes/{id}/deploy` | POST | ProcessController.java | 部署流程 |

### 7.2 流程执行接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/bpm/execution/start` | POST | ExecutionController.java | 启动流程 |
| `/api/bpm/execution/tasks` | GET | ExecutionController.java | 获取我的任务 |
| `/api/bpm/execution/tasks/{id}/complete` | POST | ExecutionController.java | 完成任务 |
| `/api/bpm/execution/tasks/{id}/delegate` | POST | ExecutionController.java | 委派任务 |

### 7.3 流程监控接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/bpm/monitor/instance/{id}` | GET | MonitorController.java | 获取流程状态 |
| `/api/bpm/monitor/history/{instanceId}` | GET | MonitorController.java | 获取流程历史 |
| `/api/bpm/monitor/instance/{id}/suspend` | POST | MonitorController.java | 挂起流程 |
| `/api/bpm/monitor/instance/{id}/terminate` | POST | MonitorController.java | 终止流程 |

### 7.4 流程报表接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/bpm/report/statistics` | GET | ReportController.java | 流程统计 |
| `/api/bpm/report/efficiency` | GET | ReportController.java | 效率分析 |
| `/api/bpm/report/kpi` | GET | ReportController.java | KPI 报表 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 流程设计 | process:read, process:write | 查看和创建流程 |
| 流程执行 | execution:read, execution:write | 启动和处理流程 |
| 流程监控 | monitor:read, monitor:write | 监控和管理流程 |

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
| OA | REST API | 办公审批流程 |
| ERP | REST API | 业务流程协同 |
| CRM | REST API | 客户流程协同 |

---

**文档结束**