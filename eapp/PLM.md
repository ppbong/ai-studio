# PLM 产品生命周期管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 PLM（Product Lifecycle Management）产品生命周期管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
PLM 系统作为企业级应用体系的产品研发管理平台，负责产品数据管理、BOM 管理、研发项目管理、文档管理等产品全生命周期管理，与 ERP、SCM 系统紧密协同。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 产品数据管理 | 产品信息、规格、参数管理 | 高 |
| 2 | BOM 管理 | 物料清单、BOM 版本、BOM 对比 | 高 |
| 3 | 研发项目管理 | 项目立项、进度跟踪、资源管理 | 高 |
| 4 | 文档管理 | 技术文档、图纸、版本控制 | 高 |
| 5 | 变更管理 | 工程变更、变更审批、变更追溯 | 高 |
| 6 | 物料管理 | 物料编码、物料认证、优选库 | 高 |
| 7 | 质量管理 | 质量问题、质量改进 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 响应时间 < 300ms |
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
| 产品模块 | 产品数据管理 | 产品信息、规格参数 |
| BOM 模块 | BOM 管理 | 物料清单、版本管理 |
| 项目模块 | 研发项目管理 | 项目立项、进度跟踪 |
| 文档模块 | 文档管理 | 技术文档、图纸 |
| 变更模块 | 变更管理 | 工程变更、审批 |
| 物料模块 | 物料管理 | 物料编码、认证 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/plm/
  │   │   │   ├── controller/
  │   │   │   │   ├── ProductController.java     # 产品管理
  │   │   │   │   ├── BomController.java         # BOM 管理
  │   │   │   │   ├── ProjectController.java     # 项目管理
  │   │   │   │   ├── DocumentController.java    # 文档管理
  │   │   │   │   └── ChangeController.java      # 变更管理
  │   │   │   ├── service/
  │   │   │   │   ├── ProductService.java
  │   │   │   │   ├── BomService.java
  │   │   │   │   ├── ProjectService.java
  │   │   │   │   ├── DocumentService.java
  │   │   │   │   └── ChangeService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── PlmApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── bom/                               # BOM 组件
  │   │   │   ├── BomTree.vue
  │   │   │   └── BomCompare.vue
  │   ├── views/
  │   │   ├── product/                           # 产品管理
  │   │   │   └── index.vue
  │   │   ├── bom/                               # BOM 管理
  │   │   │   └── index.vue
  │   │   ├── project/                           # 项目管理
  │   │   │   └── index.vue
  │   │   ├── document/                          # 文档管理
  │   │   │   └── index.vue
  │   │   └── change/                            # 变更管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ProductService (产品服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createProduct` | 创建产品 | `ProductCreateRequest request` | `ProductResponse` | 抛出`BusinessException` |
| `updateProduct` | 更新产品 | `Long productId, ProductUpdateRequest request` | `ProductResponse` | 抛出`ProductNotFoundException` |
| `getProducts` | 查询产品列表 | `ProductSearchRequest request` | `Page<ProductResponse>` | - |

#### 5.1.2 BomService (BOM 服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createBom` | 创建 BOM | `BomCreateRequest request` | `BomResponse` | 抛出`BusinessException` |
| `getBomVersions` | 获取 BOM 版本 | `Long productId` | `List<BomVersionResponse>` | 抛出`ProductNotFoundException` |
| `compareBom` | BOM 对比 | `Long bomId1, Long bomId2` | `BomCompareResult` | 抛出`BomNotFoundException` |

#### 5.1.3 ChangeService (变更服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createChangeRequest` | 创建变更请求 | `ChangeRequestCreateRequest request` | `ChangeRequestResponse` | 抛出`BusinessException` |
| `approveChange` | 审批变更 | `Long changeRequestId, ApproveRequest request` | `ChangeRequestResponse` | 抛出`ChangeRequestNotFoundException` |
| `executeChange` | 执行变更 | `Long changeRequestId` | `ChangeRequestResponse` | 抛出`ChangeRequestNotFoundException` |

### 5.2 DTO 结构定义

**ProductCreateRequest（创建产品请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| productCode | String | 产品编码 | 非空，唯一 |
| productName | String | 产品名称 | 非空 |
| category | String | 产品分类 | 非空 |
| specifications | String | 规格参数 (JSON) | 可选 |
| version | String | 版本号 | 非空 |

**BomItemRequest（BOM 明细请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| materialId | Long | 物料 ID | 非空 |
| materialCode | String | 物料编码 | 非空 |
| quantity | BigDecimal | 数量 | 非空 |
| unit | String | 单位 | 非空 |
| level | Integer | 层级 | 非空 |
| children | List<BomItemRequest> | 子项 | 可选 |

**ChangeRequestCreateRequest（创建变更请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| changeType | String | 变更类型 | 非空 |
| targetId | Long | 目标 ID | 非空 |
| reason | String | 变更原因 | 非空 |
| description | String | 变更描述 | 非空 |
| priority | String | 优先级 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 产品表 (plm_product)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 产品 ID |
| product_code | VARCHAR(50) | UNIQUE, NOT NULL | 产品编码 |
| product_name | VARCHAR(200) | NOT NULL | 产品名称 |
| category | VARCHAR(50) | NOT NULL | 产品分类 |
| specifications | TEXT | - | 规格参数 (JSON) |
| version | VARCHAR(20) | NOT NULL | 版本号 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 BOM 表 (plm_bom)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | BOM ID |
| product_id | BIGINT | FOREIGN KEY, NOT NULL | 产品 ID |
| version | VARCHAR(20) | NOT NULL | 版本号 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| content | TEXT | NOT NULL | BOM 内容 (JSON) |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 物料表 (plm_material)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 物料 ID |
| material_code | VARCHAR(50) | UNIQUE, NOT NULL | 物料编码 |
| material_name | VARCHAR(200) | NOT NULL | 物料名称 |
| category | VARCHAR(50) | NOT NULL | 物料分类 |
| specification | VARCHAR(500) | - | 规格型号 |
| unit | VARCHAR(20) | NOT NULL | 单位 |
| status | VARCHAR(20) | DEFAULT 'ACTIVE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 研发项目表 (plm_project)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 项目 ID |
| project_code | VARCHAR(50) | UNIQUE, NOT NULL | 项目编号 |
| project_name | VARCHAR(200) | NOT NULL | 项目名称 |
| description | TEXT | - | 项目描述 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| start_date | DATE | NOT NULL | 开始日期 |
| end_date | DATE | - | 结束日期 |
| manager_id | BIGINT | NOT NULL | 项目经理 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 变更请求表 (plm_change_request)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 变更 ID |
| change_no | VARCHAR(50) | UNIQUE, NOT NULL | 变更编号 |
| change_type | VARCHAR(50) | NOT NULL | 变更类型 |
| target_type | VARCHAR(50) | NOT NULL | 目标类型 |
| target_id | BIGINT | NOT NULL | 目标 ID |
| reason | VARCHAR(500) | NOT NULL | 变更原因 |
| description | TEXT | NOT NULL | 变更描述 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.6 技术文档表 (plm_document)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 文档 ID |
| doc_code | VARCHAR(50) | UNIQUE, NOT NULL | 文档编码 |
| title | VARCHAR(200) | NOT NULL | 文档标题 |
| category | VARCHAR(50) | NOT NULL | 文档分类 |
| file_path | VARCHAR(500) | NOT NULL | 文件路径 |
| version | VARCHAR(20) | NOT NULL | 版本号 |
| status | VARCHAR(20) | DEFAULT 'PUBLISHED' | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 产品管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/plm/products` | GET | ProductController.java | 查询产品列表 |
| `/api/plm/products/{id}` | GET | ProductController.java | 查询产品详情 |
| `/api/plm/products` | POST | ProductController.java | 创建产品 |
| `/api/plm/products/{id}` | PUT | ProductController.java | 更新产品 |

### 7.2 BOM 管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/plm/boms` | GET | BomController.java | 查询 BOM 列表 |
| `/api/plm/boms/{id}` | GET | BomController.java | 查询 BOM 详情 |
| `/api/plm/boms` | POST | BomController.java | 创建 BOM |
| `/api/plm/boms/compare` | GET | BomController.java | BOM 对比 |

### 7.3 变更管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/plm/changes` | GET | ChangeController.java | 查询变更列表 |
| `/api/plm/changes` | POST | ChangeController.java | 创建变更请求 |
| `/api/plm/changes/{id}/approve` | POST | ChangeController.java | 审批变更 |
| `/api/plm/changes/{id}/execute` | POST | ChangeController.java | 执行变更 |

### 7.4 项目管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/plm/projects` | GET | ProjectController.java | 查询项目列表 |
| `/api/plm/projects` | POST | ProjectController.java | 创建项目 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 产品管理 | product:read, product:write | 查看和编辑产品 |
| BOM 管理 | bom:read, bom:write | 查看和编辑 BOM |
| 变更管理 | change:read, change:approve | 查看和审批变更 |

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
| ERP | REST API + 消息队列 | BOM 同步、物料同步 |
| SCM | REST API | 物料信息同步 |

---

**文档结束**