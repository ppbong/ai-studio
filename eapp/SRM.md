# SRM 供应商关系管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 SRM（Supplier Relationship Management）供应商关系管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
SRM 系统作为企业级应用体系的供应商关系管理平台，负责供应商准入、供应商评估、采购协同、供应商绩效等供应商管理业务，与 SCM、ERP、FMS 系统紧密协同，实现供应商全生命周期管理。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 供应商准入 | 供应商注册、资质审核、准入审批 | 高 |
| 2 | 供应商档案 | 供应商信息、资质证书、联系人 | 高 |
| 3 | 供应商评估 | 定期评估、评分管理、评级管理 | 高 |
| 4 | 采购协同 | 采购订单协同、交货协同、对账协同 | 高 |
| 5 | 供应商绩效 | 交货绩效、质量绩效、服务绩效 | 高 |
| 6 | 供应商门户 | 供应商自助服务、订单查询、对账查询 | 中 |
| 7 | 供应商分析 | 供应商分析、风险预警、成本分析 | 中 |

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
| 准入模块 | 供应商准入 | 注册、审核、审批 |
| 档案模块 | 供应商档案 | 信息、资质、联系人 |
| 评估模块 | 供应商评估 | 定期评估、评分 |
| 协同模块 | 采购协同 | 订单、交货、对账 |
| 绩效模块 | 供应商绩效 | 交货、质量、服务 |
| 门户模块 | 供应商门户 | 自助服务 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/srm/
  │   │   │   ├── controller/
  │   │   │   │   ├── SupplierController.java   # 供应商管理
  │   │   │   │   ├── EvaluationController.java  # 供应商评估
  │   │   │   │   ├── CollaborationController.java # 采购协同
  │   │   │   │   ├── PerformanceController.java # 供应商绩效
  │   │   │   │   └── PortalController.java     # 供应商门户
  │   │   │   ├── service/
  │   │   │   │   ├── SupplierService.java
  │   │   │   │   ├── EvaluationService.java
  │   │   │   │   ├── CollaborationService.java
  │   │   │   │   ├── PerformanceService.java
  │   │   │   │   └── PortalService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── SrmApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── scorecard/                        # 评分卡组件
  │   │   │   └── ScoreCard.vue
  │   ├── views/
  │   │   ├── supplier/                        # 供应商管理
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── evaluation/                      # 供应商评估
  │   │   │   └── index.vue
  │   │   ├── collaboration/                   # 采购协同
  │   │   │   └── index.vue
  │   │   ├── performance/                     # 供应商绩效
  │   │   │   └── index.vue
  │   │   └── analysis/                        # 供应商分析
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 SupplierService (供应商服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `registerSupplier` | 供应商注册 | `SupplierRegisterRequest request` | `SupplierResponse` | 抛出`BusinessException` |
| `approveSupplier` | 审批供应商 | `Long supplierId, ApprovalRequest request` | `SupplierResponse` | 抛出`SupplierNotFoundException` |
| `updateSupplier` | 更新供应商信息 | `Long supplierId, SupplierUpdateRequest request` | `SupplierResponse` | 抛出`SupplierNotFoundException` |
| `getSuppliers` | 查询供应商列表 | `SupplierSearchRequest request` | `Page<SupplierResponse>` | - |

#### 5.1.2 EvaluationService (评估服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createEvaluation` | 创建评估 | `EvaluationCreateRequest request` | `EvaluationResponse` | 抛出`BusinessException` |
| `submitScore` | 提交评分 | `Long evaluationId, ScoreRequest request` | `ScoreResponse` | 抛出`EvaluationNotFoundException` |
| `calculateRating` | 计算评级 | `Long supplierId` | `RatingResponse` | - |

#### 5.1.3 CollaborationService (协同服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `syncPurchaseOrder` | 同步采购订单 | `PurchaseOrderSyncRequest request` | `SyncResponse` | 抛出`BusinessException` |
| `confirmDelivery` | 确认交货 | `Long orderId, DeliveryConfirmRequest request` | `DeliveryResponse` | 抛出`OrderNotFoundException` |
| `reconcileAccount` | 对账 | `ReconciliationRequest request` | `ReconciliationResponse` | - |

### 5.2 DTO 结构定义

**SupplierRegisterRequest（供应商注册请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| supplierName | String | 供应商名称 | 非空 |
| supplierType | String | 供应商类型 | 非空 |
| businessLicense | String | 营业执照号 | 非空 |
| taxId | String | 税号 | 非空 |
| contactPerson | String | 联系人 | 非空 |
| contactPhone | String | 联系电话 | 非空 |
| contactEmail | String | 联系邮箱 | 非空 |
| address | String | 地址 | 非空 |

**EvaluationCreateRequest（创建评估请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| supplierId | Long | 供应商 ID | 非空 |
| evaluationType | String | 评估类型 | 非空 |
| evaluationPeriod | String | 评估周期 | 非空 |
| evaluatorId | Long | 评估人 ID | 非空 |
| criteria | List<CriteriaRequest> | 评估标准 | 非空 |

**PerformanceResponse（绩效响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| supplierId | Long | 供应商 ID |
| supplierName | String | 供应商名称 |
| deliveryScore | BigDecimal | 交货评分 |
| qualityScore | BigDecimal | 质量评分 |
| serviceScore | BigDecimal | 服务评分 |
| totalScore | BigDecimal | 总评分 |
| rating | String | 评级 |
| evaluatedAt | LocalDateTime | 评估时间 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 供应商表 (srm_supplier)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 供应商 ID |
| supplier_no | VARCHAR(50) | UNIQUE, NOT NULL | 供应商编号 |
| supplier_name | VARCHAR(200) | NOT NULL | 供应商名称 |
| supplier_type | VARCHAR(50) | NOT NULL | 供应商类型 |
| business_license | VARCHAR(100) | NOT NULL | 营业执照号 |
| tax_id | VARCHAR(50) | NOT NULL | 税号 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| rating | VARCHAR(20) | - | 评级 |
| contact_person | VARCHAR(100) | NOT NULL | 联系人 |
| contact_phone | VARCHAR(20) | NOT NULL | 联系电话 |
| contact_email | VARCHAR(100) | NOT NULL | 联系邮箱 |
| address | VARCHAR(500) | NOT NULL | 地址 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 供应商资质表 (srm_supplier_qualification)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 资质 ID |
| supplier_id | BIGINT | FOREIGN KEY, NOT NULL | 供应商 ID |
| qualification_type | VARCHAR(50) | NOT NULL | 资质类型 |
| certificate_no | VARCHAR(100) | NOT NULL | 证书编号 |
| issue_date | DATE | NOT NULL | 发证日期 |
| expiry_date | DATE | NOT NULL | 有效期 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| file_url | VARCHAR(500) | - | 证书文件 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 供应商评估表 (srm_supplier_evaluation)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 评估 ID |
| supplier_id | BIGINT | FOREIGN KEY, NOT NULL | 供应商 ID |
| evaluation_type | VARCHAR(50) | NOT NULL | 评估类型 |
| evaluation_period | VARCHAR(50) | NOT NULL | 评估周期 |
| evaluator_id | BIGINT | NOT NULL | 评估人 ID |
| total_score | DECIMAL(5,2) | NOT NULL | 总评分 |
| rating | VARCHAR(20) | NOT NULL | 评级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| evaluated_at | DATETIME | NOT NULL | 评估时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 供应商绩效表 (srm_supplier_performance)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 绩效 ID |
| supplier_id | BIGINT | FOREIGN KEY, NOT NULL | 供应商 ID |
| period | VARCHAR(50) | NOT NULL | 统计周期 |
| delivery_score | DECIMAL(5,2) | NOT NULL | 交货评分 |
| quality_score | DECIMAL(5,2) | NOT NULL | 质量评分 |
| service_score | DECIMAL(5,2) | NOT NULL | 服务评分 |
| total_score | DECIMAL(5,2) | NOT NULL | 总评分 |
| rating | VARCHAR(20) | NOT NULL | 评级 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 采购协同表 (srm_purchase_collaboration)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 协同 ID |
| purchase_order_id | BIGINT | NOT NULL | 采购订单 ID |
| supplier_id | BIGINT | FOREIGN KEY, NOT NULL | 供应商 ID |
| collaboration_type | VARCHAR(50) | NOT NULL | 协同类型 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| confirmed_at | DATETIME | - | 确认时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 供应商管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/srm/suppliers` | GET | SupplierController.java | 查询供应商列表 |
| `/api/srm/suppliers/{id}` | GET | SupplierController.java | 查询供应商详情 |
| `/api/srm/suppliers/register` | POST | SupplierController.java | 供应商注册 |
| `/api/srm/suppliers/{id}/approve` | POST | SupplierController.java | 审批供应商 |
| `/api/srm/suppliers/{id}` | PUT | SupplierController.java | 更新供应商信息 |

### 7.2 供应商评估接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/srm/evaluations` | GET | EvaluationController.java | 查询评估列表 |
| `/api/srm/evaluations` | POST | EvaluationController.java | 创建评估 |
| `/api/srm/evaluations/{id}/score` | POST | EvaluationController.java | 提交评分 |
| `/api/srm/suppliers/{id}/rating` | GET | EvaluationController.java | 计算评级 |

### 7.3 采购协同接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/srm/collaboration/orders` | POST | CollaborationController.java | 同步采购订单 |
| `/api/srm/collaboration/delivery` | POST | CollaborationController.java | 确认交货 |
| `/api/srm/collaboration/reconcile` | POST | CollaborationController.java | 对账 |

### 7.4 供应商绩效接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/srm/performance/{supplierId}` | GET | PerformanceController.java | 查询供应商绩效 |
| `/api/srm/performance/analysis` | GET | PerformanceController.java | 绩效分析 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制
- 供应商门户使用独立的认证机制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 供应商管理 | supplier:read, supplier:write | 查看和管理供应商 |
| 供应商评估 | evaluation:read, evaluation:write | 查看和创建评估 |
| 采购协同 | collaboration:read, collaboration:write | 查看和协同订单 |

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
| SCM | REST API + 消息队列 | 采购订单协同 |
| ERP | REST API | 供应商信息同步 |
| FMS | REST API | 付款协同 |

---

**文档结束**