# ESP 企业服务平台系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 ESP（Enterprise Service Platform）企业服务平台系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
ESP 系统作为企业级应用体系的统一服务平台，负责服务目录管理、服务编排、服务监控和服务治理，与各业务系统紧密协同，实现企业服务的统一管理和高效交付。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 服务目录 | 服务注册、服务发现、服务目录 | 高 |
| 2 | 服务编排 | 服务组合、流程编排、API 网关 | 高 |
| 3 | 服务监控 | 服务监控、性能分析、故障诊断 | 高 |
| 4 | 服务治理 | 服务管理、版本管理、权限控制 | 高 |
| 5 | API 管理 | API 设计、API 发布、API 安全 | 高 |
| 6 | 服务集成 | 系统集成、数据同步、消息队列 | 中 |
| 7 | 服务分析 | 服务调用分析、性能报告、成本分析 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 响应时间 < 50ms，支持高并发 |
| 可用性 | 99.99% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **API 网关模式**: 统一入口，路由转发

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 目录模块 | 服务目录 | 注册、发现、目录 |
| 编排模块 | 服务编排 | 组合、流程、网关 |
| 监控模块 | 服务监控 | 监控、分析、诊断 |
| 治理模块 | 服务治理 | 管理、版本、权限 |
| API模块 | API管理 | 设计、发布、安全 |
| 集成模块 | 服务集成 | 集成、同步、消息 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/esp/
  │   │   │   ├── controller/
  │   │   │   │   ├── ServiceCatalogController.java # 服务目录
  │   │   │   │   ├── ServiceOrchestrationController.java # 服务编排
  │   │   │   │   ├── ServiceMonitorController.java # 服务监控
  │   │   │   │   ├── APIController.java           # API管理
  │   │   │   │   └── ServiceGovernanceController.java # 服务治理
  │   │   │   ├── service/
  │   │   │   │   ├── ServiceCatalogService.java
  │   │   │   │   ├── ServiceOrchestrationService.java
  │   │   │   │   ├── ServiceMonitorService.java
  │   │   │   │   ├── APIService.java
  │   │   │   │   └── ServiceGovernanceService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── EspApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── service/                         # 服务组件
  │   │   │   └── ServiceCard.vue
  │   ├── views/
  │   │   ├── catalog/                         # 服务目录
  │   │   │   └── index.vue
  │   │   ├── orchestration/                   # 服务编排
  │   │   │   └── index.vue
  │   │   ├── monitor/                         # 服务监控
  │   │   │   └── index.vue
  │   │   ├── api/                             # API管理
  │   │   │   └── index.vue
  │   │   └── governance/                      # 服务治理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ServiceCatalogService (服务目录服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `registerService` | 注册服务 | `ServiceRegisterRequest request` | `ServiceResponse` | 抛出`BusinessException` |
| `discoverService` | 发现服务 | `ServiceDiscoverRequest request` | `List<ServiceResponse>` | - |
| `getServiceDetail` | 获取服务详情 | `String serviceId` | `ServiceDetailResponse` | 抛出`ServiceNotFoundException` |
| `getServices` | 查询服务列表 | `ServiceSearchRequest request` | `Page<ServiceResponse>` | - |

#### 5.1.2 ServiceOrchestrationService (服务编排服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createComposition` | 创建服务组合 | `CompositionCreateRequest request` | `CompositionResponse` | 抛出`BusinessException` |
| `executeComposition` | 执行服务组合 | `Long compositionId` | `CompositionExecutionResponse` | 抛出`CompositionNotFoundException` |
| `getCompositions` | 查询服务组合 | `CompositionSearchRequest request` | `Page<CompositionResponse>` | - |

#### 5.1.3 ServiceMonitorService (服务监控服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `getServiceMetrics` | 获取服务指标 | `String serviceId` | `ServiceMetricsResponse` | 抛出`ServiceNotFoundException` |
| `getServiceHealth` | 获取服务健康状态 | `String serviceId` | `ServiceHealthResponse` | 抛出`ServiceNotFoundException` |
| `getServiceLogs` | 获取服务日志 | `ServiceLogRequest request` | `List<ServiceLogResponse>` | - |

### 5.2 DTO 结构定义

**ServiceRegisterRequest（服务注册请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| serviceName | String | 服务名称 | 非空 |
| serviceCode | String | 服务编码 | 非空，唯一 |
| serviceType | String | 服务类型 | 非空 |
| endpoint | String | 服务端点 | 非空 |
| version | String | 版本号 | 非空 |
| description | String | 描述 | 可选 |

**CompositionCreateRequest（创建服务组合请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| compositionName | String | 组合名称 | 非空 |
| description | String | 描述 | 可选 |
| steps | List<CompositionStep> | 组合步骤 | 非空 |
| timeout | Integer | 超时时间(秒) | 可选 |

**ServiceMetricsResponse（服务指标响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| serviceId | String | 服务 ID |
| serviceName | String | 服务名称 |
| requestCount | Long | 请求次数 |
| successRate | Double | 成功率 |
| avgResponseTime | Long | 平均响应时间(ms) |
| peakResponseTime | Long | 峰值响应时间(ms) |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 服务注册表 (esp_service_registry)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | VARCHAR(50) | PRIMARY KEY | 服务 ID |
| service_name | VARCHAR(200) | NOT NULL | 服务名称 |
| service_code | VARCHAR(50) | UNIQUE, NOT NULL | 服务编码 |
| service_type | VARCHAR(50) | NOT NULL | 服务类型 |
| endpoint | VARCHAR(500) | NOT NULL | 服务端点 |
| version | VARCHAR(20) | NOT NULL | 版本号 |
| description | VARCHAR(500) | - | 描述 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| metadata | TEXT | - | 元数据(JSON) |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 服务组合表 (esp_service_composition)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 组合 ID |
| composition_name | VARCHAR(100) | NOT NULL | 组合名称 |
| description | VARCHAR(500) | - | 描述 |
| steps | TEXT | NOT NULL | 组合步骤(JSON) |
| timeout | INT | DEFAULT 300 | 超时时间(秒) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 服务指标表 (esp_service_metrics)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 指标 ID |
| service_id | VARCHAR(50) | FOREIGN KEY, NOT NULL | 服务 ID |
| metric_name | VARCHAR(50) | NOT NULL | 指标名称 |
| metric_value | DECIMAL(15,2) | NOT NULL | 指标值 |
| timestamp | DATETIME | NOT NULL | 时间戳 |

#### 6.1.4 API 定义表 (esp_api_definition)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | API ID |
| api_name | VARCHAR(100) | NOT NULL | API 名称 |
| api_path | VARCHAR(200) | NOT NULL | API 路径 |
| http_method | VARCHAR(10) | NOT NULL | HTTP 方法 |
| service_id | VARCHAR(50) | FOREIGN KEY, NOT NULL | 服务 ID |
| version | VARCHAR(20) | NOT NULL | 版本号 |
| request_schema | TEXT | - | 请求 Schema |
| response_schema | TEXT | - | 响应 Schema |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 服务调用记录表 (esp_service_invocation)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 调用 ID |
| service_id | VARCHAR(50) | NOT NULL | 服务 ID |
| api_id | BIGINT | FOREIGN KEY | API ID |
| request_id | VARCHAR(50) | NOT NULL | 请求 ID |
| status_code | INT | NOT NULL | 状态码 |
| response_time | INT | NOT NULL | 响应时间(ms) |
| occurred_at | DATETIME | NOT NULL | 发生时间 |

---

## 7. API 接口设计

### 7.1 服务目录接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/esp/services` | GET | ServiceCatalogController.java | 查询服务列表 |
| `/api/esp/services/{id}` | GET | ServiceCatalogController.java | 查询服务详情 |
| `/api/esp/services` | POST | ServiceCatalogController.java | 注册服务 |
| `/api/esp/services/discover` | POST | ServiceCatalogController.java | 发现服务 |

### 7.2 服务编排接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/esp/compositions` | GET | ServiceOrchestrationController.java | 查询服务组合 |
| `/api/esp/compositions` | POST | ServiceOrchestrationController.java | 创建服务组合 |
| `/api/esp/compositions/{id}/execute` | POST | ServiceOrchestrationController.java | 执行服务组合 |

### 7.3 服务监控接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/esp/monitor/metrics/{serviceId}` | GET | ServiceMonitorController.java | 获取服务指标 |
| `/api/esp/monitor/health/{serviceId}` | GET | ServiceMonitorController.java | 获取服务健康状态 |
| `/api/esp/monitor/logs` | GET | ServiceMonitorController.java | 获取服务日志 |

### 7.4 API 管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/esp/api` | GET | APIController.java | 查询 API 列表 |
| `/api/esp/api` | POST | APIController.java | 创建 API |
| `/api/esp/api/{id}/publish` | POST | APIController.java | 发布 API |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 服务管理 | service:read, service:write | 查看和管理服务 |
| API 管理 | api:read, api:write | 查看和管理 API |
| 监控管理 | monitor:read | 查看监控数据 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| Spring Cloud Gateway | 4.1.x | API 网关 |
| Prometheus | 2.4.x | 监控 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| 各业务系统 | REST API | 服务注册和调用 |
| BI | REST API | 监控数据可视化 |

---

**文档结束**
