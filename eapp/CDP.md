# CDP 客户数据平台系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 CDP（Customer Data Platform）客户数据平台系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
CDP 系统作为企业级应用体系的客户数据管理核心平台，负责客户数据采集、整合、分析和应用，与 CRM、MA、BI 系统紧密协同，实现全渠道客户数据的统一管理和精准营销。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 数据采集 | 多渠道数据采集、实时数据同步 | 高 |
| 2 | 数据整合 | 客户身份识别、数据融合、统一视图 | 高 |
| 3 | 客户画像 | 客户标签、行为分析、人群细分 | 高 |
| 4 | 数据应用 | 数据导出、API 服务、营销触达 | 高 |
| 5 | 数据治理 | 数据质量、合规管理、数据安全 | 高 |
| 6 | 分析洞察 | 客户分析、趋势分析、预测分析 | 中 |
| 7 | 营销自动化 | 营销活动触发、个性化推荐 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 支持百万级客户数据，查询响应 < 100ms |
| 可用性 | 99.9% 高可用 |
| 安全性 | 符合 GDPR、等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **事件驱动**: 通过消息队列实现实时数据同步

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 采集模块 | 数据采集 | 多渠道、实时同步 |
| 整合模块 | 数据整合 | 身份识别、数据融合 |
| 画像模块 | 客户画像 | 标签、行为、细分 |
| 应用模块 | 数据应用 | 导出、API、触达 |
| 治理模块 | 数据治理 | 质量、合规、安全 |
| 分析模块 | 分析洞察 | 分析、预测 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/cdp/
  │   │   │   ├── controller/
  │   │   │   │   ├── DataCollectionController.java # 数据采集
  │   │   │   │   ├── CustomerController.java       # 客户管理
  │   │   │   │   ├── ProfileController.java        # 客户画像
  │   │   │   │   ├── ApplicationController.java    # 数据应用
  │   │   │   │   └── GovernanceController.java     # 数据治理
  │   │   │   ├── service/
  │   │   │   │   ├── DataCollectionService.java
  │   │   │   │   ├── CustomerService.java
  │   │   │   │   ├── ProfileService.java
  │   │   │   │   ├── ApplicationService.java
  │   │   │   │   └── GovernanceService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── CdpApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── customer/                         # 客户组件
  │   │   │   └── CustomerProfile.vue
  │   ├── views/
  │   │   ├── dashboard/                       # 数据看板
  │   │   │   └── index.vue
  │   │   ├── customer/                        # 客户管理
  │   │   │   └── index.vue
  │   │   ├── profile/                         # 客户画像
  │   │   │   └── index.vue
  │   │   ├── segment/                         # 人群细分
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

#### 5.1.1 DataCollectionService (数据采集服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `collectData` | 采集数据 | `DataCollectRequest request` | `DataCollectResponse` | 抛出`CollectException` |
| `batchCollect` | 批量采集 | `List<DataCollectRequest> requests` | `List<DataCollectResponse>` | - |
| `syncData` | 数据同步 | `DataSyncRequest request` | `DataSyncResponse` | 抛出`SyncException` |

#### 5.1.2 CustomerService (客户服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `identifyCustomer` | 客户身份识别 | `CustomerIdentifyRequest request` | `CustomerResponse` | 抛出`BusinessException` |
| `getCustomerProfile` | 获取客户画像 | `Long customerId` | `CustomerProfileResponse` | 抛出`CustomerNotFoundException` |
| `getCustomers` | 查询客户列表 | `CustomerSearchRequest request` | `Page<CustomerResponse>` | - |

#### 5.1.3 ProfileService (画像服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `addTag` | 添加标签 | `TagAddRequest request` | `TagResponse` | 抛出`BusinessException` |
| `createSegment` | 创建人群细分 | `SegmentCreateRequest request` | `SegmentResponse` | 抛出`BusinessException` |
| `getSegments` | 查询人群细分 | `SegmentSearchRequest request` | `Page<SegmentResponse>` | - |

### 5.2 DTO 结构定义

**DataCollectRequest（数据采集请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| sourceType | String | 来源类型 | 非空 |
| sourceId | String | 来源标识 | 非空 |
| customerId | String | 客户标识 | 可选 |
| eventType | String | 事件类型 | 非空 |
| properties | Map<String, Object> | 属性数据 | 可选 |

**CustomerIdentifyRequest（客户身份识别请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| identifiers | List<Identifier> | 身份标识列表 | 非空 |
| attributes | Map<String, Object> | 属性数据 | 可选 |

**SegmentCreateRequest（创建人群细分请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| segmentName | String | 细分名称 | 非空 |
| description | String | 描述 | 可选 |
| filters | List<Filter> | 过滤条件 | 非空 |
| refreshType | String | 刷新类型 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 客户表 (cdp_customer)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 客户 ID |
| customer_id | VARCHAR(50) | UNIQUE, NOT NULL | 客户唯一标识 |
| first_name | VARCHAR(100) | - | 名 |
| last_name | VARCHAR(100) | - | 姓 |
| email | VARCHAR(200) | - | 邮箱 |
| phone | VARCHAR(50) | - | 手机号 |
| gender | VARCHAR(10) | - | 性别 |
| birth_date | DATE | - | 出生日期 |
| attributes | TEXT | - | 属性数据(JSON) |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 客户身份标识表 (cdp_customer_identifier)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 标识 ID |
| customer_id | BIGINT | FOREIGN KEY, NOT NULL | 客户 ID |
| identifier_type | VARCHAR(50) | NOT NULL | 标识类型 |
| identifier_value | VARCHAR(200) | NOT NULL | 标识值 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 客户标签表 (cdp_customer_tag)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 标签 ID |
| customer_id | BIGINT | FOREIGN KEY, NOT NULL | 客户 ID |
| tag_name | VARCHAR(100) | NOT NULL | 标签名称 |
| tag_value | VARCHAR(200) | - | 标签值 |
| tag_category | VARCHAR(50) | NOT NULL | 标签类别 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 客户事件表 (cdp_customer_event)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 事件 ID |
| customer_id | BIGINT | FOREIGN KEY | 客户 ID |
| event_type | VARCHAR(50) | NOT NULL | 事件类型 |
| event_name | VARCHAR(100) | NOT NULL | 事件名称 |
| source_type | VARCHAR(50) | NOT NULL | 来源类型 |
| properties | TEXT | - | 事件属性(JSON) |
| occurred_at | DATETIME | NOT NULL | 发生时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 人群细分表 (cdp_segment)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 细分 ID |
| segment_name | VARCHAR(100) | NOT NULL | 细分名称 |
| description | VARCHAR(500) | - | 描述 |
| filters | TEXT | NOT NULL | 过滤条件(JSON) |
| refresh_type | VARCHAR(20) | NOT NULL | 刷新类型 |
| member_count | INT | DEFAULT 0 | 成员数量 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 数据采集接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cdp/collect` | POST | DataCollectionController.java | 采集数据 |
| `/api/cdp/collect/batch` | POST | DataCollectionController.java | 批量采集 |
| `/api/cdp/sync` | POST | DataCollectionController.java | 数据同步 |

### 7.2 客户管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cdp/customers` | GET | CustomerController.java | 查询客户列表 |
| `/api/cdp/customers/{id}` | GET | CustomerController.java | 查询客户详情 |
| `/api/cdp/customers/identify` | POST | CustomerController.java | 客户身份识别 |

### 7.3 客户画像接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cdp/profile/{customerId}` | GET | ProfileController.java | 获取客户画像 |
| `/api/cdp/profile/tags` | POST | ProfileController.java | 添加标签 |
| `/api/cdp/segments` | GET | ProfileController.java | 查询人群细分 |
| `/api/cdp/segments` | POST | ProfileController.java | 创建人群细分 |

### 7.4 数据治理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cdp/governance/quality` | GET | GovernanceController.java | 数据质量报告 |
| `/api/cdp/governance/compliance` | GET | GovernanceController.java | 合规检查 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 数据采集 | collect:read, collect:write | 查看和采集数据 |
| 客户管理 | customer:read, customer:write | 查看和管理客户 |
| 画像管理 | profile:read, profile:write | 查看和管理画像 |

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
| Elasticsearch | 8.x | 客户检索 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| CRM | REST API + 消息队列 | 客户数据同步 |
| MA | REST API + 消息队列 | 营销自动化 |
| BI | REST API | 数据可视化 |

---

**文档结束**
