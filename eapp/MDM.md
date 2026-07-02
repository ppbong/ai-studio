# MDM 主数据管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 MDM（Master Data Management）主数据管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
MDM 系统作为企业级应用体系的主数据管理平台，负责主数据的创建、维护、分发和治理，与 ERP、CRM、SCM 等系统紧密协同，确保企业核心数据的一致性和准确性。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 主数据建模 | 数据模型定义、属性配置、关系定义 | 高 |
| 2 | 主数据管理 | 数据创建、编辑、审核、发布 | 高 |
| 3 | 数据质量管理 | 数据校验、数据清洗、数据标准化 | 高 |
| 4 | 数据分发 | 数据同步、数据订阅、数据接口 | 高 |
| 5 | 数据治理 | 数据血缘、数据审计、数据权限 | 高 |
| 6 | 数据版本 | 版本管理、变更追踪、历史回溯 | 中 |
| 7 | 主数据报表 | 数据统计、质量报告、使用分析 | 中 |

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
- **事件驱动**: 通过消息队列实现数据同步

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 建模模块 | 主数据建模 | 数据模型、属性、关系 |
| 管理模块 | 主数据管理 | 创建、编辑、审核 |
| 质量模块 | 数据质量管理 | 校验、清洗、标准化 |
| 分发模块 | 数据分发 | 同步、订阅、接口 |
| 治理模块 | 数据治理 | 血缘、审计、权限 |
| 版本模块 | 数据版本 | 版本管理、变更追踪 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/mdm/
  │   │   │   ├── controller/
  │   │   │   │   ├── ModelController.java      # 数据建模
  │   │   │   │   ├── DataController.java       # 主数据管理
  │   │   │   │   ├── QualityController.java    # 数据质量
  │   │   │   │   └── DistributionController.java # 数据分发
  │   │   │   ├── service/
  │   │   │   │   ├── ModelService.java
  │   │   │   │   ├── MasterDataService.java
  │   │   │   │   ├── QualityService.java
  │   │   │   │   └── DistributionService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── MdmApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── model/                           # 建模组件
  │   │   │   └── ModelEditor.vue
  │   ├── views/
  │   │   ├── model/                           # 数据建模
  │   │   │   └── index.vue
  │   │   ├── data/                            # 主数据管理
  │   │   │   ├── list.vue
  │   │   │   └── edit.vue
  │   │   ├── quality/                         # 数据质量
  │   │   │   └── index.vue
  │   │   ├── distribution/                    # 数据分发
  │   │   │   └── index.vue
  │   │   └── governance/                      # 数据治理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ModelService (数据建模服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createModel` | 创建数据模型 | `ModelCreateRequest request` | `ModelResponse` | 抛出`BusinessException` |
| `updateModel` | 更新数据模型 | `Long modelId, ModelUpdateRequest request` | `ModelResponse` | 抛出`ModelNotFoundException` |
| `getModel` | 获取数据模型 | `Long modelId` | `ModelResponse` | 抛出`ModelNotFoundException` |
| `getModels` | 查询模型列表 | `ModelSearchRequest request` | `Page<ModelResponse>` | - |

#### 5.1.2 MasterDataService (主数据服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createMasterData` | 创建主数据 | `MasterDataCreateRequest request` | `MasterDataResponse` | 抛出`BusinessException` |
| `updateMasterData` | 更新主数据 | `Long modelId, Long dataId, MasterDataUpdateRequest request` | `MasterDataResponse` | 抛出`DataNotFoundException` |
| `approveMasterData` | 审核主数据 | `Long modelId, Long dataId` | `MasterDataResponse` | 抛出`DataNotFoundException` |
| `getMasterData` | 查询主数据 | `Long modelId, Long dataId` | `MasterDataResponse` | 抛出`DataNotFoundException` |

#### 5.1.3 QualityService (数据质量服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `validateData` | 数据校验 | `Long modelId, Map<String, Object> data` | `ValidationResponse` | - |
| `cleanData` | 数据清洗 | `Long modelId, Long dataId` | `CleanResponse` | 抛出`DataNotFoundException` |
| `standardizeData` | 数据标准化 | `Long modelId, Long dataId` | `StandardizeResponse` | 抛出`DataNotFoundException` |

### 5.2 DTO 结构定义

**ModelCreateRequest（创建模型请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| modelName | String | 模型名称 | 非空 |
| modelCode | String | 模型编码 | 非空，唯一 |
| description | String | 描述 | 可选 |
| attributes | List<AttributeRequest> | 属性列表 | 非空 |
| relationships | List<RelationshipRequest> | 关系定义 | 可选 |

**MasterDataCreateRequest（创建主数据请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| modelId | Long | 模型 ID | 非空 |
| dataCode | String | 数据编码 | 非空，唯一 |
| attributes | Map<String, Object> | 属性数据 | 非空 |

**ValidationResponse（校验响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| valid | Boolean | 是否校验通过 |
| errors | List<ValidationError> | 校验错误列表 |
| warnings | List<ValidationWarning> | 校验警告列表 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 数据模型表 (mdm_model)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 模型 ID |
| model_name | VARCHAR(100) | NOT NULL | 模型名称 |
| model_code | VARCHAR(50) | UNIQUE, NOT NULL | 模型编码 |
| description | VARCHAR(500) | - | 描述 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 模型属性表 (mdm_model_attribute)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 属性 ID |
| model_id | BIGINT | FOREIGN KEY, NOT NULL | 模型 ID |
| attribute_name | VARCHAR(100) | NOT NULL | 属性名称 |
| attribute_code | VARCHAR(50) | NOT NULL | 属性编码 |
| data_type | VARCHAR(20) | NOT NULL | 数据类型 |
| length | INT | - | 长度 |
| is_required | BOOLEAN | DEFAULT FALSE | 是否必填 |
| validation_rule | VARCHAR(200) | - | 校验规则 |
| sort_order | INT | DEFAULT 0 | 排序 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 主数据表 (mdm_master_data)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 数据 ID |
| model_id | BIGINT | FOREIGN KEY, NOT NULL | 模型 ID |
| data_code | VARCHAR(50) | NOT NULL | 数据编码 |
| data_name | VARCHAR(200) | NOT NULL | 数据名称 |
| attributes | TEXT | NOT NULL | 属性数据(JSON) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| version | INT | DEFAULT 1 | 版本号 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.4 数据变更记录表 (mdm_data_change)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 变更 ID |
| model_id | BIGINT | NOT NULL | 模型 ID |
| data_id | BIGINT | NOT NULL | 数据 ID |
| change_type | VARCHAR(20) | NOT NULL | 变更类型 |
| old_value | TEXT | - | 旧值(JSON) |
| new_value | TEXT | - | 新值(JSON) |
| changed_by | BIGINT | NOT NULL | 变更人 ID |
| changed_at | DATETIME | NOT NULL | 变更时间 |

---

## 7. API 接口设计

### 7.1 数据建模接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/mdm/models` | GET | ModelController.java | 查询模型列表 |
| `/api/mdm/models/{id}` | GET | ModelController.java | 查询模型详情 |
| `/api/mdm/models` | POST | ModelController.java | 创建数据模型 |
| `/api/mdm/models/{id}` | PUT | ModelController.java | 更新数据模型 |

### 7.2 主数据管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/mdm/data/{modelId}` | GET | DataController.java | 查询主数据列表 |
| `/api/mdm/data/{modelId}/{dataId}` | GET | DataController.java | 查询主数据详情 |
| `/api/mdm/data/{modelId}` | POST | DataController.java | 创建主数据 |
| `/api/mdm/data/{modelId}/{dataId}` | PUT | DataController.java | 更新主数据 |
| `/api/mdm/data/{modelId}/{dataId}/approve` | POST | DataController.java | 审核主数据 |

### 7.3 数据质量接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/mdm/quality/validate` | POST | QualityController.java | 数据校验 |
| `/api/mdm/quality/clean/{modelId}/{dataId}` | POST | QualityController.java | 数据清洗 |
| `/api/mdm/quality/standardize/{modelId}/{dataId}` | POST | QualityController.java | 数据标准化 |

### 7.4 数据分发接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/mdm/distribution/subscribe` | POST | DistributionController.java | 订阅数据 |
| `/api/mdm/distribution/sync` | POST | DistributionController.java | 同步数据 |
| `/api/mdm/distribution/export/{modelId}` | GET | DistributionController.java | 导出数据 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 模型管理 | model:read, model:write | 查看和管理模型 |
| 数据管理 | data:read, data:write | 查看和管理主数据 |
| 数据分发 | distribution:read, distribution:write | 订阅和同步数据 |

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
| ERP | REST API + 消息队列 | 主数据同步 |
| CRM | REST API + 消息队列 | 客户主数据 |
| SCM | REST API + 消息队列 | 供应商主数据 |

---

**文档结束**