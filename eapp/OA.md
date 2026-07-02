# OA办公自动化系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述OA（Office Automation）办公自动化系统的设计方案，包括系统架构、功能模块、API接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
OA系统作为企业级应用体系的办公协作平台，负责流程审批、文档管理、日程安排、会议管理等日常办公事务，提升企业协同办公效率。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 流程审批 | 请假、报销、采购等审批流程 | 高 |
| 2 | 文档管理 | 文件上传、共享、版本控制 | 高 |
| 3 | 日程管理 | 个人日程、团队日程、日程提醒 | 高 |
| 4 | 会议管理 | 会议预约、会议室管理、会议纪要 | 高 |
| 5 | 通知公告 | 公司公告、部门通知 | 高 |
| 6 | 任务管理 | 任务分配、进度跟踪 | 中 |
| 7 | 办公用品管理 | 办公用品申请、领用 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 响应时间 < 200ms |
| 可用性 | 99.9%高可用 |
| 安全性 | 符合等保2.0三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **事件驱动**: 通过消息队列实现系统间解耦

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 流程模块 | 流程审批 | 工作流引擎、审批流程 |
| 文档模块 | 文档管理 | 文件存储、版本控制 |
| 日程模块 | 日程管理 | 个人/团队日程 |
| 会议模块 | 会议管理 | 会议室预约、会议纪要 |
| 公告模块 | 通知公告 | 公告发布、通知推送 |
| 任务模块 | 任务管理 | 任务分配、跟踪 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/oa/
  │   │   │   ├── controller/
  │   │   │   │   ├── WorkflowController.java  # 流程审批
  │   │   │   │   ├── DocumentController.java   # 文档管理
  │   │   │   │   ├── ScheduleController.java   # 日程管理
  │   │   │   │   ├── MeetingController.java    # 会议管理
  │   │   │   │   └── AnnouncementController.java # 公告管理
  │   │   │   ├── service/
  │   │   │   │   ├── WorkflowService.java
  │   │   │   │   ├── DocumentService.java
  │   │   │   │   ├── ScheduleService.java
  │   │   │   │   ├── MeetingService.java
  │   │   │   │   └── AnnouncementService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   ├── workflow/                    # 工作流引擎
  │   │   │   │   ├── Engine.java
  │   │   │   │   └── FlowDefinition.java
  │   │   │   └── OaApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   ├── views/
  │   │   ├── workflow/                        # 流程审批
  │   │   │   ├── apply.vue
  │   │   │   └── approve.vue
  │   │   ├── document/                        # 文档管理
  │   │   │   └── index.vue
  │   │   ├── schedule/                        # 日程管理
  │   │   │   └── index.vue
  │   │   ├── meeting/                         # 会议管理
  │   │   │   └── index.vue
  │   │   └── announcement/                    # 公告管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 WorkflowService (流程服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createWorkflow` | 创建流程实例 | `WorkflowCreateRequest request` | `WorkflowResponse` | 抛出`BusinessException` |
| `approve` | 审批流程 | `Long workflowId, ApproveRequest request` | `WorkflowResponse` | 抛出`WorkflowNotFoundException` |
| `getWorkflows` | 查询流程列表 | `WorkflowSearchRequest request` | `Page<WorkflowResponse>` | - |

#### 5.1.2 DocumentService (文档服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `uploadDocument` | 上传文档 | `MultipartFile file, DocumentMeta meta` | `DocumentResponse` | 抛出`UploadException` |
| `getDocuments` | 查询文档列表 | `DocumentSearchRequest request` | `Page<DocumentResponse>` | - |
| `downloadDocument` | 下载文档 | `Long documentId` | `Resource` | 抛出`DocumentNotFoundException` |

### 5.2 DTO结构定义

**WorkflowCreateRequest（创建流程请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| flowType | String | 流程类型 | 非空 |
| title | String | 流程标题 | 非空 |
| content | String | 流程内容 | 非空 |
| attachments | List<String> | 附件ID列表 | 可选 |

**DocumentMeta（文档元数据）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| name | String | 文档名称 | 非空 |
| category | String | 文档分类 | 可选 |
| description | String | 文档描述 | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 流程实例表 (oa_workflow)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 流程ID |
| flow_type | VARCHAR(50) | NOT NULL | 流程类型 |
| title | VARCHAR(200) | NOT NULL | 流程标题 |
| content | TEXT | NOT NULL | 流程内容 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| current_node | VARCHAR(50) | NOT NULL | 当前节点 |
| creator_id | BIGINT | NOT NULL | 创建人ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 流程审批记录表 (oa_workflow_approval)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 审批记录ID |
| workflow_id | BIGINT | FOREIGN KEY, NOT NULL | 流程ID |
| approver_id | BIGINT | NOT NULL | 审批人ID |
| status | VARCHAR(20) | NOT NULL | 审批状态 |
| comment | VARCHAR(500) | - | 审批意见 |
| approved_at | DATETIME | NOT NULL | 审批时间 |

#### 6.1.3 文档表 (oa_document)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 文档ID |
| name | VARCHAR(200) | NOT NULL | 文档名称 |
| category | VARCHAR(50) | - | 文档分类 |
| file_path | VARCHAR(500) | NOT NULL | 文件路径 |
| file_size | BIGINT | NOT NULL | 文件大小 |
| version | INT | DEFAULT 1 | 版本号 |
| created_by | BIGINT | NOT NULL | 创建人ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 日程表 (oa_schedule)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 日程ID |
| title | VARCHAR(200) | NOT NULL | 日程标题 |
| description | VARCHAR(500) | - | 日程描述 |
| start_time | DATETIME | NOT NULL | 开始时间 |
| end_time | DATETIME | NOT NULL | 结束时间 |
| location | VARCHAR(200) | - | 地点 |
| organizer_id | BIGINT | NOT NULL | 组织者ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 会议室表 (oa_meeting_room)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 会议室ID |
| name | VARCHAR(100) | NOT NULL | 会议室名称 |
| capacity | INT | NOT NULL | 容纳人数 |
| location | VARCHAR(200) | NOT NULL | 位置 |
| equipment | VARCHAR(500) | - | 设备清单 |
| status | VARCHAR(20) | DEFAULT 'AVAILABLE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API接口设计

### 7.1 流程审批接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/oa/workflows` | GET | WorkflowController.java | 查询流程列表 |
| `/api/oa/workflows/{id}` | GET | WorkflowController.java | 查询流程详情 |
| `/api/oa/workflows` | POST | WorkflowController.java | 创建流程 |
| `/api/oa/workflows/{id}/approve` | POST | WorkflowController.java | 审批流程 |

### 7.2 文档管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/oa/documents` | GET | DocumentController.java | 查询文档列表 |
| `/api/oa/documents/{id}` | GET | DocumentController.java | 下载文档 |
| `/api/oa/documents` | POST | DocumentController.java | 上传文档 |

### 7.3 日程管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/oa/schedules` | GET | ScheduleController.java | 查询日程列表 |
| `/api/oa/schedules` | POST | ScheduleController.java | 创建日程 |
| `/api/oa/schedules/{id}` | PUT | ScheduleController.java | 更新日程 |

### 7.4 会议管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/oa/meeting-rooms` | GET | MeetingController.java | 查询会议室列表 |
| `/api/oa/meetings` | POST | MeetingController.java | 创建会议 |

### 7.5 公告管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/oa/announcements` | GET | AnnouncementController.java | 查询公告列表 |
| `/api/oa/announcements` | POST | AnnouncementController.java | 发布公告 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过SSO系统进行统一身份认证
- 使用JWT令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 流程审批 | workflow:create, workflow:approve | 创建和审批流程 |
| 文档管理 | document:read, document:write | 查看和上传文档 |
| 日程管理 | schedule:read, schedule:write | 查看和创建日程 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| MinIO | 8.5+ | 文件存储 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| ERP | REST API | 报销流程与财务集成 |

---

**文档结束**