# PMS 项目管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 PMS（Project Management System）项目管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
PMS 系统作为企业级应用体系的项目管理平台，负责项目立项、项目计划、任务管理、资源分配、进度跟踪、成本控制等项目管理业务，与 ERP、OA 系统紧密协同，确保项目按时按质完成。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 项目立项 | 项目申请、审批、立项 | 高 |
| 2 | 项目计划 | WBS 分解、里程碑、甘特图 | 高 |
| 3 | 任务管理 | 任务分配、任务跟踪、任务协作 | 高 |
| 4 | 资源管理 | 人员分配、设备资源、预算分配 | 高 |
| 5 | 进度管理 | 进度跟踪、进度预警、进度报告 | 高 |
| 6 | 成本管理 | 成本预算、成本核算、成本控制 | 高 |
| 7 | 风险管理 | 风险识别、风险评估、风险应对 | 中 |

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
| 立项模块 | 项目立项 | 项目申请、审批 |
| 计划模块 | 项目计划 | WBS、里程碑 |
| 任务模块 | 任务管理 | 任务分配、跟踪 |
| 资源模块 | 资源管理 | 人员、设备、预算 |
| 进度模块 | 进度管理 | 进度跟踪、预警 |
| 成本模块 | 成本管理 | 预算、核算、控制 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/pms/
  │   │   │   ├── controller/
  │   │   │   │   ├── ProjectController.java    # 项目管理
  │   │   │   │   ├── TaskController.java       # 任务管理
  │   │   │   │   ├── ResourceController.java   # 资源管理
  │   │   │   │   ├── ScheduleController.java   # 进度管理
  │   │   │   │   └── CostController.java       # 成本管理
  │   │   │   ├── service/
  │   │   │   │   ├── ProjectService.java
  │   │   │   │   ├── TaskService.java
  │   │   │   │   ├── ResourceService.java
  │   │   │   │   ├── ScheduleService.java
  │   │   │   │   └── CostService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── PmsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── gantt/                            # 甘特图组件
  │   │   │   └── GanttChart.vue
  │   ├── views/
  │   │   ├── project/                          # 项目管理
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── task/                             # 任务管理
  │   │   │   ├── list.vue
  │   │   │   └── kanban.vue
  │   │   ├── resource/                         # 资源管理
  │   │   │   └── index.vue
  │   │   ├── schedule/                         # 进度管理
  │   │   │   └── index.vue
  │   │   └── cost/                             # 成本管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ProjectService (项目服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createProject` | 创建项目 | `ProjectCreateRequest request` | `ProjectResponse` | 抛出`BusinessException` |
| `updateProject` | 更新项目 | `Long projectId, ProjectUpdateRequest request` | `ProjectResponse` | 抛出`ProjectNotFoundException` |
| `closeProject` | 结项 | `Long projectId` | `ProjectResponse` | 抛出`ProjectNotFoundException` |
| `getProjects` | 查询项目列表 | `ProjectSearchRequest request` | `Page<ProjectResponse>` | - |

#### 5.1.2 TaskService (任务服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createTask` | 创建任务 | `TaskCreateRequest request` | `TaskResponse` | 抛出`BusinessException` |
| `assignTask` | 分配任务 | `Long taskId, Long assigneeId` | `TaskResponse` | 抛出`TaskNotFoundException` |
| `completeTask` | 完成任务 | `Long taskId` | `TaskResponse` | 抛出`TaskNotFoundException` |
| `getTasks` | 查询任务列表 | `TaskSearchRequest request` | `Page<TaskResponse>` | - |

#### 5.1.3 ScheduleService (进度服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `updateProgress` | 更新进度 | `Long projectId, ProgressUpdateRequest request` | `ProgressResponse` | 抛出`ProjectNotFoundException` |
| `getGanttData` | 获取甘特图数据 | `Long projectId` | `GanttDataResponse` | 抛出`ProjectNotFoundException` |
| `getMilestones` | 获取里程碑 | `Long projectId` | `List<MilestoneResponse>` | 抛出`ProjectNotFoundException` |

### 5.2 DTO 结构定义

**ProjectCreateRequest（创建项目请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| projectNo | String | 项目编号 | 非空，唯一 |
| projectName | String | 项目名称 | 非空 |
| description | String | 项目描述 | 可选 |
| startDate | LocalDate | 开始日期 | 非空 |
| endDate | LocalDate | 结束日期 | 非空 |
| managerId | Long | 项目经理 ID | 非空 |
| budget | BigDecimal | 预算金额 | 非空 |

**TaskCreateRequest（创建任务请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| projectId | Long | 项目 ID | 非空 |
| taskName | String | 任务名称 | 非空 |
| description | String | 任务描述 | 可选 |
| parentId | Long | 父任务 ID | 可选 |
| assigneeId | Long | 负责人 ID | 可选 |
| startDate | LocalDate | 开始日期 | 非空 |
| endDate | LocalDate | 结束日期 | 非空 |
| priority | String | 优先级 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 项目表 (pms_project)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 项目 ID |
| project_no | VARCHAR(50) | UNIQUE, NOT NULL | 项目编号 |
| project_name | VARCHAR(200) | NOT NULL | 项目名称 |
| description | TEXT | - | 项目描述 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| start_date | DATE | NOT NULL | 开始日期 |
| end_date | DATE | NOT NULL | 结束日期 |
| actual_start_date | DATE | - | 实际开始日期 |
| actual_end_date | DATE | - | 实际结束日期 |
| manager_id | BIGINT | NOT NULL | 项目经理 ID |
| budget | DECIMAL(15,2) | NOT NULL | 预算金额 |
| actual_cost | DECIMAL(15,2) | DEFAULT 0 | 实际成本 |
| progress | DECIMAL(5,2) | DEFAULT 0 | 进度百分比 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 任务表 (pms_task)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 任务 ID |
| project_id | BIGINT | FOREIGN KEY, NOT NULL | 项目 ID |
| task_name | VARCHAR(200) | NOT NULL | 任务名称 |
| description | TEXT | - | 任务描述 |
| parent_id | BIGINT | FOREIGN KEY | 父任务 ID |
| assignee_id | BIGINT | - | 负责人 ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| start_date | DATE | NOT NULL | 开始日期 |
| end_date | DATE | NOT NULL | 结束日期 |
| progress | DECIMAL(5,2) | DEFAULT 0 | 进度百分比 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.3 里程碑表 (pms_milestone)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 里程碑 ID |
| project_id | BIGINT | FOREIGN KEY, NOT NULL | 项目 ID |
| name | VARCHAR(200) | NOT NULL | 里程碑名称 |
| planned_date | DATE | NOT NULL | 计划日期 |
| actual_date | DATE | - | 实际日期 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 项目成员表 (pms_project_member)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 成员 ID |
| project_id | BIGINT | FOREIGN KEY, NOT NULL | 项目 ID |
| user_id | BIGINT | NOT NULL | 用户 ID |
| role | VARCHAR(50) | NOT NULL | 角色 |
| join_date | DATE | NOT NULL | 加入日期 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 项目风险表 (pms_risk)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 风险 ID |
| project_id | BIGINT | FOREIGN KEY, NOT NULL | 项目 ID |
| risk_name | VARCHAR(200) | NOT NULL | 风险名称 |
| description | TEXT | NOT NULL | 风险描述 |
| probability | VARCHAR(20) | NOT NULL | 发生概率 |
| impact | VARCHAR(20) | NOT NULL | 影响程度 |
| mitigation_plan | TEXT | - | 应对措施 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 项目管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/pms/projects` | GET | ProjectController.java | 查询项目列表 |
| `/api/pms/projects/{id}` | GET | ProjectController.java | 查询项目详情 |
| `/api/pms/projects` | POST | ProjectController.java | 创建项目 |
| `/api/pms/projects/{id}` | PUT | ProjectController.java | 更新项目 |
| `/api/pms/projects/{id}/close` | POST | ProjectController.java | 结项 |

### 7.2 任务管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/pms/tasks` | GET | TaskController.java | 查询任务列表 |
| `/api/pms/tasks` | POST | TaskController.java | 创建任务 |
| `/api/pms/tasks/{id}/assign` | POST | TaskController.java | 分配任务 |
| `/api/pms/tasks/{id}/complete` | POST | TaskController.java | 完成任务 |

### 7.3 进度管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/pms/schedule/gantt/{projectId}` | GET | ScheduleController.java | 获取甘特图数据 |
| `/api/pms/schedule/progress` | POST | ScheduleController.java | 更新进度 |
| `/api/pms/milestones` | GET | ScheduleController.java | 获取里程碑 |

### 7.4 资源管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/pms/resources/members` | GET | ResourceController.java | 查询项目成员 |
| `/api/pms/resources/members` | POST | ResourceController.java | 添加项目成员 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 项目管理 | project:read, project:write | 查看和编辑项目 |
| 任务管理 | task:read, task:write | 查看和编辑任务 |
| 进度管理 | schedule:read, schedule:write | 查看和更新进度 |

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
| ERP | REST API + 消息队列 | 项目成本同步 |
| OA | REST API | 项目审批流程 |

---

**文档结束**