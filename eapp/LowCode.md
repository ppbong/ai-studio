# LowCode 低代码平台系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 LowCode 低代码平台系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
LowCode 系统作为企业级应用体系的快速开发平台，负责应用构建、表单设计、流程编排、数据管理等低代码开发业务，与 BPM、OA、ESP 系统紧密协同，实现业务应用的快速构建和部署。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 应用构建 | 应用创建、页面设计、组件配置 | 高 |
| 2 | 表单设计 | 表单设计、字段配置、校验规则 | 高 |
| 3 | 流程编排 | 流程设计、节点配置、条件分支 | 高 |
| 4 | 数据管理 | 数据源配置、数据建模、数据绑定 | 高 |
| 5 | 权限管理 | 角色权限、数据权限、操作权限 | 高 |
| 6 | 应用发布 | 应用部署、版本管理、环境切换 | 中 |
| 7 | 扩展开发 | 自定义组件、自定义逻辑、API 集成 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 页面加载 < 500ms，支持高并发 |
| 可用性 | 99.9% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **可视化设计**: 拖拽式设计，所见即所得

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 构建模块 | 应用构建 | 创建、设计、配置 |
| 表单模块 | 表单设计 | 设计、字段、校验 |
| 流程模块 | 流程编排 | 设计、节点、分支 |
| 数据模块 | 数据管理 | 数据源、建模、绑定 |
| 权限模块 | 权限管理 | 角色、数据、操作 |
| 发布模块 | 应用发布 | 部署、版本、环境 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/lowcode/
  │   │   │   ├── controller/
  │   │   │   │   ├── AppController.java         # 应用管理
  │   │   │   │   ├── PageController.java        # 页面管理
  │   │   │   │   ├── FormController.java        # 表单管理
  │   │   │   │   ├── WorkflowController.java    # 流程管理
  │   │   │   │   ├── DataSourceController.java  # 数据源管理
  │   │   │   │   └── PermissionController.java  # 权限管理
  │   │   │   ├── service/
  │   │   │   │   ├── AppService.java
  │   │   │   │   ├── PageService.java
  │   │   │   │   ├── FormService.java
  │   │   │   │   ├── WorkflowService.java
  │   │   │   │   ├── DataSourceService.java
  │   │   │   │   └── PermissionService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── LowCodeApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── designer/                        # 设计器组件
  │   │   │   └── PageDesigner.vue
  │   ├── views/
  │   │   ├── apps/                            # 应用管理
  │   │   │   └── index.vue
  │   │   ├── pages/                           # 页面设计
  │   │   │   └── index.vue
  │   │   ├── forms/                           # 表单设计
  │   │   │   └── index.vue
  │   │   ├── workflows/                       # 流程设计
  │   │   │   └── index.vue
  │   │   └── datasources/                     # 数据源管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 AppService (应用服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createApp` | 创建应用 | `AppCreateRequest request` | `AppResponse` | 抛出`BusinessException` |
| `updateApp` | 更新应用 | `Long appId, AppUpdateRequest request` | `AppResponse` | 抛出`AppNotFoundException` |
| `publishApp` | 发布应用 | `Long appId` | `AppResponse` | 抛出`AppNotFoundException` |
| `getApps` | 查询应用列表 | `AppSearchRequest request` | `Page<AppResponse>` | - |

#### 5.1.2 PageService (页面服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createPage` | 创建页面 | `PageCreateRequest request` | `PageResponse` | 抛出`BusinessException` |
| `updatePage` | 更新页面 | `Long pageId, PageUpdateRequest request` | `PageResponse` | 抛出`PageNotFoundException` |
| `getPageDetail` | 获取页面详情 | `Long pageId` | `PageDetailResponse` | 抛出`PageNotFoundException` |
| `getPages` | 查询页面列表 | `PageSearchRequest request` | `Page<PageResponse>` | - |

#### 5.1.3 FormService (表单服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createForm` | 创建表单 | `FormCreateRequest request` | `FormResponse` | 抛出`BusinessException` |
| `updateForm` | 更新表单 | `Long formId, FormUpdateRequest request` | `FormResponse` | 抛出`FormNotFoundException` |
| `submitForm` | 提交表单 | `Long formId, FormSubmitRequest request` | `FormSubmitResponse` | 抛出`FormNotFoundException` |
| `getForms` | 查询表单列表 | `FormSearchRequest request` | `Page<FormResponse>` | - |

### 5.2 DTO 结构定义

**AppCreateRequest（创建应用请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| appName | String | 应用名称 | 非空 |
| appCode | String | 应用编码 | 非空，唯一 |
| description | String | 描述 | 可选 |
| icon | String | 应用图标 | 可选 |

**PageCreateRequest（创建页面请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| appId | Long | 应用 ID | 非空 |
| pageName | String | 页面名称 | 非空 |
| pageType | String | 页面类型 | 非空 |
| layout | String | 布局配置(JSON) | 非空 |
| components | String | 组件配置(JSON) | 非空 |

**FormCreateRequest（创建表单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| appId | Long | 应用 ID | 非空 |
| formName | String | 表单名称 | 非空 |
| fields | String | 字段配置(JSON) | 非空 |
| validation | String | 校验规则(JSON) | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 应用表 (lowcode_app)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 应用 ID |
| app_name | VARCHAR(100) | NOT NULL | 应用名称 |
| app_code | VARCHAR(50) | UNIQUE, NOT NULL | 应用编码 |
| description | VARCHAR(500) | - | 描述 |
| icon | VARCHAR(200) | - | 应用图标 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| version | VARCHAR(20) | DEFAULT '1.0' | 版本号 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 页面表 (lowcode_page)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 页面 ID |
| app_id | BIGINT | FOREIGN KEY, NOT NULL | 应用 ID |
| page_name | VARCHAR(100) | NOT NULL | 页面名称 |
| page_type | VARCHAR(50) | NOT NULL | 页面类型 |
| layout | TEXT | NOT NULL | 布局配置(JSON) |
| components | TEXT | NOT NULL | 组件配置(JSON) |
| route_path | VARCHAR(200) | - | 路由路径 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 表单表 (lowcode_form)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 表单 ID |
| app_id | BIGINT | FOREIGN KEY, NOT NULL | 应用 ID |
| form_name | VARCHAR(100) | NOT NULL | 表单名称 |
| fields | TEXT | NOT NULL | 字段配置(JSON) |
| validation | TEXT | - | 校验规则(JSON) |
| data_source | VARCHAR(50) | - | 数据源 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 表单数据表 (lowcode_form_data)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 数据 ID |
| form_id | BIGINT | FOREIGN KEY, NOT NULL | 表单 ID |
| data | TEXT | NOT NULL | 表单数据(JSON) |
| submitted_by | BIGINT | - | 提交人 ID |
| submitted_at | DATETIME | NOT NULL | 提交时间 |

#### 6.1.5 数据源表 (lowcode_data_source)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 数据源 ID |
| app_id | BIGINT | FOREIGN KEY, NOT NULL | 应用 ID |
| source_name | VARCHAR(100) | NOT NULL | 数据源名称 |
| source_type | VARCHAR(50) | NOT NULL | 数据源类型 |
| connection_config | TEXT | NOT NULL | 连接配置(JSON) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 应用管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/lowcode/apps` | GET | AppController.java | 查询应用列表 |
| `/api/lowcode/apps/{id}` | GET | AppController.java | 查询应用详情 |
| `/api/lowcode/apps` | POST | AppController.java | 创建应用 |
| `/api/lowcode/apps/{id}/publish` | POST | AppController.java | 发布应用 |

### 7.2 页面管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/lowcode/pages` | GET | PageController.java | 查询页面列表 |
| `/api/lowcode/pages/{id}` | GET | PageController.java | 查询页面详情 |
| `/api/lowcode/pages` | POST | PageController.java | 创建页面 |
| `/api/lowcode/pages/{id}` | PUT | PageController.java | 更新页面 |

### 7.3 表单管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/lowcode/forms` | GET | FormController.java | 查询表单列表 |
| `/api/lowcode/forms/{id}` | GET | FormController.java | 查询表单详情 |
| `/api/lowcode/forms` | POST | FormController.java | 创建表单 |
| `/api/lowcode/forms/{id}/submit` | POST | FormController.java | 提交表单 |

### 7.4 流程管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/lowcode/workflows` | GET | WorkflowController.java | 查询流程列表 |
| `/api/lowcode/workflows` | POST | WorkflowController.java | 创建流程 |
| `/api/lowcode/workflows/{id}/start` | POST | WorkflowController.java | 启动流程 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 应用管理 | app:read, app:write | 查看和管理应用 |
| 页面管理 | page:read, page:write | 查看和设计页面 |
| 表单管理 | form:read, form:write | 查看和设计表单 |

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
| BPM | REST API | 流程集成 |
| OA | REST API | 应用集成 |
| ESP | REST API | 服务集成 |

---

**文档结束**