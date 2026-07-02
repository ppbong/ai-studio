# FMS 财务管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 FMS（Financial Management System）财务管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
FMS 系统作为企业级应用体系的专业财务管理平台，负责总账管理、应收应付、成本核算、预算管理、资金管理等财务业务，与 ERP、CRM、SCM 系统紧密协同，确保企业财务数据的准确性和合规性。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 总账管理 | 会计科目、会计分录、账簿管理 | 高 |
| 2 | 应收管理 | 应收账款、开票管理、收款核销 | 高 |
| 3 | 应付管理 | 应付账款、付款管理、付款核销 | 高 |
| 4 | 成本核算 | 成本计算、成本分析、成本控制 | 高 |
| 5 | 预算管理 | 预算编制、预算控制、预算分析 | 高 |
| 6 | 资金管理 | 资金计划、资金调度、资金监控 | 高 |
| 7 | 财务报表 | 资产负债表、利润表、现金流量表 | 高 |

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
| 总账模块 | 总账管理 | 会计科目、分录、账簿 |
| 应收模块 | 应收管理 | 应收、开票、收款 |
| 应付模块 | 应付管理 | 应付、付款、核销 |
| 成本模块 | 成本核算 | 成本计算、分析 |
| 预算模块 | 预算管理 | 预算编制、控制 |
| 资金模块 | 资金管理 | 资金计划、调度 |
| 报表模块 | 财务报表 | 资产负债表、利润表 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/fms/
  │   │   │   ├── controller/
  │   │   │   │   ├── GeneralLedgerController.java  # 总账管理
  │   │   │   │   ├── AccountsReceivableController.java # 应收管理
  │   │   │   │   ├── AccountsPayableController.java    # 应付管理
  │   │   │   │   ├── CostController.java              # 成本核算
  │   │   │   │   ├── BudgetController.java            # 预算管理
  │   │   │   │   └── FundController.java              # 资金管理
  │   │   │   ├── service/
  │   │   │   │   ├── GeneralLedgerService.java
  │   │   │   │   ├── AccountsReceivableService.java
  │   │   │   │   ├── AccountsPayableService.java
  │   │   │   │   ├── CostService.java
  │   │   │   │   ├── BudgetService.java
  │   │   │   │   └── FundService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── FmsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   ├── views/
  │   │   ├── ledger/                            # 总账管理
  │   │   │   └── index.vue
  │   │   ├── receivable/                        # 应收管理
  │   │   │   └── index.vue
  │   │   ├── payable/                           # 应付管理
  │   │   │   └── index.vue
  │   │   ├── cost/                              # 成本核算
  │   │   │   └── index.vue
  │   │   ├── budget/                            # 预算管理
  │   │   │   └── index.vue
  │   │   ├── fund/                              # 资金管理
  │   │   │   └── index.vue
  │   │   └── report/                            # 财务报表
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 GeneralLedgerService (总账服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createAccountEntry` | 创建会计分录 | `AccountEntryCreateRequest request` | `AccountEntryResponse` | 抛出`BusinessException` |
| `generateLedger` | 生成账簿 | `String period` | `LedgerResponse` | 抛出`BusinessException` |
| `getAccountEntries` | 查询会计分录 | `AccountEntrySearchRequest request` | `Page<AccountEntryResponse>` | - |

#### 5.1.2 AccountsReceivableService (应收服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createInvoice` | 创建发票 | `InvoiceCreateRequest request` | `InvoiceResponse` | 抛出`BusinessException` |
| `receivePayment` | 收款核销 | `ReceivePaymentRequest request` | `PaymentResponse` | 抛出`InvoiceNotFoundException` |
| `getReceivables` | 查询应收账款 | `ReceivableSearchRequest request` | `Page<ReceivableResponse>` | - |

#### 5.1.3 CostService (成本服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `calculateCost` | 成本计算 | `CostCalculateRequest request` | `CostResponse` | 抛出`BusinessException` |
| `getCostAnalysis` | 成本分析 | `CostAnalysisRequest request` | `CostAnalysisResponse` | - |

### 5.2 DTO 结构定义

**AccountEntryCreateRequest（创建会计分录请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| accountId | Long | 科目 ID | 非空 |
| amount | BigDecimal | 金额 | 非空 |
| direction | String | 方向(DEBIT/CREDIT) | 非空 |
| date | LocalDate | 日期 | 非空 |
| description | String | 摘要 | 可选 |

**InvoiceCreateRequest（创建发票请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| invoiceNo | String | 发票号 | 非空 |
| customerId | Long | 客户 ID | 非空 |
| amount | BigDecimal | 金额 | 非空 |
| taxAmount | BigDecimal | 税额 | 非空 |
| invoiceDate | LocalDate | 开票日期 | 非空 |
| dueDate | LocalDate | 到期日期 | 非空 |

**CostCalculateRequest（成本计算请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| productId | Long | 产品 ID | 非空 |
| period | String | 核算期间 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 会计科目表 (fms_account)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 科目 ID |
| code | VARCHAR(50) | UNIQUE, NOT NULL | 科目编码 |
| name | VARCHAR(100) | NOT NULL | 科目名称 |
| type | VARCHAR(20) | NOT NULL | 科目类型 |
| parent_id | BIGINT | FOREIGN KEY | 父科目 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 会计分录表 (fms_account_entry)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 分录 ID |
| account_id | BIGINT | FOREIGN KEY, NOT NULL | 科目 ID |
| amount | DECIMAL(15,2) | NOT NULL | 金额 |
| direction | VARCHAR(10) | NOT NULL | 借贷方向 |
| date | DATE | NOT NULL | 日期 |
| description | VARCHAR(255) | - | 摘要 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 发票表 (fms_invoice)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 发票 ID |
| invoice_no | VARCHAR(50) | UNIQUE, NOT NULL | 发票号 |
| customer_id | BIGINT | NOT NULL | 客户 ID |
| amount | DECIMAL(15,2) | NOT NULL | 金额 |
| tax_amount | DECIMAL(15,2) | NOT NULL | 税额 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| invoice_date | DATE | NOT NULL | 开票日期 |
| due_date | DATE | NOT NULL | 到期日期 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 收款记录表 (fms_receipt)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 收款 ID |
| invoice_id | BIGINT | FOREIGN KEY | 发票 ID |
| customer_id | BIGINT | NOT NULL | 客户 ID |
| amount | DECIMAL(15,2) | NOT NULL | 收款金额 |
| payment_method | VARCHAR(50) | NOT NULL | 付款方式 |
| receipt_date | DATE | NOT NULL | 收款日期 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 预算表 (fms_budget)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 预算 ID |
| budget_no | VARCHAR(50) | UNIQUE, NOT NULL | 预算编号 |
| department_id | BIGINT | NOT NULL | 部门 ID |
| category | VARCHAR(50) | NOT NULL | 预算类别 |
| amount | DECIMAL(15,2) | NOT NULL | 预算金额 |
| period | VARCHAR(20) | NOT NULL | 预算期间 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 总账管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/fms/accounts` | GET | GeneralLedgerController.java | 查询会计科目 |
| `/api/fms/entries` | GET | GeneralLedgerController.java | 查询会计分录 |
| `/api/fms/entries` | POST | GeneralLedgerController.java | 创建会计分录 |

### 7.2 应收管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/fms/invoices` | GET | AccountsReceivableController.java | 查询发票列表 |
| `/api/fms/invoices` | POST | AccountsReceivableController.java | 创建发票 |
| `/api/fms/receipts` | POST | AccountsReceivableController.java | 收款核销 |

### 7.3 应付管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/fms/payables` | GET | AccountsPayableController.java | 查询应付账款 |
| `/api/fms/payments` | POST | AccountsPayableController.java | 付款核销 |

### 7.4 成本核算接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/fms/cost/calculate` | POST | CostController.java | 成本计算 |
| `/api/fms/cost/analysis` | GET | CostController.java | 成本分析 |

### 7.5 财务报表接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/fms/reports/balance-sheet` | GET | GeneralLedgerController.java | 获取资产负债表 |
| `/api/fms/reports/income-statement` | GET | GeneralLedgerController.java | 获取利润表 |
| `/api/fms/reports/cash-flow` | GET | GeneralLedgerController.java | 获取现金流量表 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 总账管理 | ledger:read, ledger:write | 查看和创建分录 |
| 应收管理 | receivable:read, receivable:write | 查看和处理应收 |
| 应付管理 | payable:read, payable:write | 查看和处理应付 |
| 预算管理 | budget:read, budget:write | 查看和编制预算 |

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
| CRM | REST API + 消息队列 | 销售订单→开票→收款 |
| SCM | REST API + 消息队列 | 采购订单→付款 |
| ERP | REST API | 数据同步 |

---

**文档结束**