# CPM 企业绩效管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 CPM（Corporate Performance Management）企业绩效管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
CPM 系统作为企业级应用体系的绩效管理核心平台，负责战略管理、预算管理、绩效评估、报表分析等企业绩效管理业务，与 BI、FMS、ERP 系统紧密协同，实现企业战略目标的分解、执行和监控。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 战略管理 | 战略规划、目标分解、战略地图 | 高 |
| 2 | 预算管理 | 预算编制、预算执行、预算分析 | 高 |
| 3 | 绩效评估 | KPI 管理、绩效评分、绩效排名 | 高 |
| 4 | 报表分析 | 管理报表、分析模型、数据钻取 | 高 |
| 5 | 平衡计分卡 | BSC 设计、指标跟踪、战略执行 | 高 |
| 6 | 预测分析 | 财务预测、销售预测、趋势分析 | 中 |
| 7 | 协同管理 | 目标协同、任务协同、进度跟踪 | 中 |

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
| 战略模块 | 战略管理 | 规划、分解、地图 |
| 预算模块 | 预算管理 | 编制、执行、分析 |
| 绩效模块 | 绩效评估 | KPI、评分、排名 |
| 报表模块 | 报表分析 | 管理报表、分析 |
| 计分卡模块 | 平衡计分卡 | BSC、跟踪 |
| 预测模块 | 预测分析 | 财务、销售预测 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/cpm/
  │   │   │   ├── controller/
  │   │   │   │   ├── StrategyController.java   # 战略管理
  │   │   │   │   ├── BudgetController.java     # 预算管理
  │   │   │   │   ├── PerformanceController.java # 绩效评估
  │   │   │   │   ├── ReportController.java     # 报表分析
  │   │   │   │   └── ScorecardController.java  # 平衡计分卡
  │   │   │   ├── service/
  │   │   │   │   ├── StrategyService.java
  │   │   │   │   ├── BudgetService.java
  │   │   │   │   ├── PerformanceService.java
  │   │   │   │   ├── ReportService.java
  │   │   │   │   └── ScorecardService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── CpmApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── scorecard/                        # 计分卡组件
  │   │   │   └── ScoreCard.vue
  │   ├── views/
  │   │   ├── strategy/                        # 战略管理
  │   │   │   └── index.vue
  │   │   ├── budget/                          # 预算管理
  │   │   │   └── index.vue
  │   │   ├── performance/                     # 绩效评估
  │   │   │   └── index.vue
  │   │   ├── report/                          # 报表分析
  │   │   │   └── index.vue
  │   │   └── scorecard/                       # 平衡计分卡
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 StrategyService (战略服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createStrategy` | 创建战略 | `StrategyCreateRequest request` | `StrategyResponse` | 抛出`BusinessException` |
| `decomposeGoals` | 分解目标 | `Long strategyId, GoalDecomposeRequest request` | `StrategyResponse` | 抛出`StrategyNotFoundException` |
| `getStrategyMap` | 获取战略地图 | `Long strategyId` | `StrategyMapResponse` | 抛出`StrategyNotFoundException` |
| `getStrategies` | 查询战略列表 | `StrategySearchRequest request` | `Page<StrategyResponse>` | - |

#### 5.1.2 BudgetService (预算服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createBudget` | 创建预算 | `BudgetCreateRequest request` | `BudgetResponse` | 抛出`BusinessException` |
| `executeBudget` | 执行预算 | `Long budgetId, BudgetExecutionRequest request` | `BudgetResponse` | 抛出`BudgetNotFoundException` |
| `analyzeBudget` | 分析预算 | `Long budgetId` | `BudgetAnalysisResponse` | 抛出`BudgetNotFoundException` |
| `getBudgets` | 查询预算列表 | `BudgetSearchRequest request` | `Page<BudgetResponse>` | - |

#### 5.1.3 PerformanceService (绩效服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createKPI` | 创建 KPI | `KPICreateRequest request` | `KPIResponse` | 抛出`BusinessException` |
| `evaluatePerformance` | 评估绩效 | `Long kpiId, PerformanceEvaluateRequest request` | `PerformanceResponse` | 抛出`KPINotFoundException` |
| `getPerformanceRanking` | 获取绩效排名 | `PerformanceRankingRequest request` | `List<PerformanceRankingResponse>` | - |
| `getKPIs` | 查询 KPI 列表 | `KPISearchRequest request` | `Page<KPIResponse>` | - |

### 5.2 DTO 结构定义

**StrategyCreateRequest（创建战略请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| strategyName | String | 战略名称 | 非空 |
| strategyPeriod | String | 战略周期 | 非空 |
| description | String | 战略描述 | 可选 |
| goals | List<GoalRequest> | 战略目标 | 非空 |

**BudgetCreateRequest（创建预算请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| budgetName | String | 预算名称 | 非空 |
| budgetPeriod | String | 预算周期 | 非空 |
| budgetType | String | 预算类型 | 非空 |
| departmentId | Long | 部门 ID | 非空 |
| budgetAmount | BigDecimal | 预算金额 | 非空 |

**KPICreateRequest（创建 KPI 请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| kpiName | String | KPI 名称 | 非空 |
| kpiCode | String | KPI 编码 | 非空，唯一 |
| category | String | KPI 类别 | 非空 |
| targetValue | BigDecimal | 目标值 | 非空 |
| weight | BigDecimal | 权重 | 非空 |
| measurementUnit | String | 计量单位 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 战略表 (cpm_strategy)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 战略 ID |
| strategy_name | VARCHAR(200) | NOT NULL | 战略名称 |
| strategy_period | VARCHAR(50) | NOT NULL | 战略周期 |
| description | VARCHAR(500) | - | 战略描述 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 战略目标表 (cpm_strategy_goal)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 目标 ID |
| strategy_id | BIGINT | FOREIGN KEY, NOT NULL | 战略 ID |
| goal_name | VARCHAR(200) | NOT NULL | 目标名称 |
| goal_level | INT | NOT NULL | 目标层级 |
| parent_id | BIGINT | FOREIGN KEY | 父目标 ID |
| target_value | DECIMAL(15,2) | - | 目标值 |
| weight | DECIMAL(5,2) | - | 权重 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 预算表 (cpm_budget)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 预算 ID |
| budget_name | VARCHAR(200) | NOT NULL | 预算名称 |
| budget_period | VARCHAR(50) | NOT NULL | 预算周期 |
| budget_type | VARCHAR(50) | NOT NULL | 预算类型 |
| department_id | BIGINT | NOT NULL | 部门 ID |
| budget_amount | DECIMAL(15,2) | NOT NULL | 预算金额 |
| actual_amount | DECIMAL(15,2) | DEFAULT 0 | 实际金额 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 KPI 表 (cpm_kpi)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | KPI ID |
| kpi_name | VARCHAR(200) | NOT NULL | KPI 名称 |
| kpi_code | VARCHAR(50) | UNIQUE, NOT NULL | KPI 编码 |
| category | VARCHAR(50) | NOT NULL | KPI 类别 |
| target_value | DECIMAL(15,2) | NOT NULL | 目标值 |
| actual_value | DECIMAL(15,2) | DEFAULT 0 | 实际值 |
| weight | DECIMAL(5,2) | NOT NULL | 权重 |
| measurement_unit | VARCHAR(20) | NOT NULL | 计量单位 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 绩效评估表 (cpm_performance_evaluation)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 评估 ID |
| kpi_id | BIGINT | FOREIGN KEY, NOT NULL | KPI ID |
| employee_id | BIGINT | NOT NULL | 员工 ID |
| department_id | BIGINT | NOT NULL | 部门 ID |
| period | VARCHAR(50) | NOT NULL | 评估周期 |
| score | DECIMAL(5,2) | NOT NULL | 得分 |
| rating | VARCHAR(20) | NOT NULL | 评级 |
| evaluator_id | BIGINT | NOT NULL | 评估人 ID |
| evaluated_at | DATETIME | NOT NULL | 评估时间 |

---

## 7. API 接口设计

### 7.1 战略管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cpm/strategies` | GET | StrategyController.java | 查询战略列表 |
| `/api/cpm/strategies/{id}` | GET | StrategyController.java | 查询战略详情 |
| `/api/cpm/strategies` | POST | StrategyController.java | 创建战略 |
| `/api/cpm/strategies/{id}/decompose` | POST | StrategyController.java | 分解目标 |
| `/api/cpm/strategies/{id}/map` | GET | StrategyController.java | 获取战略地图 |

### 7.2 预算管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cpm/budgets` | GET | BudgetController.java | 查询预算列表 |
| `/api/cpm/budgets/{id}` | GET | BudgetController.java | 查询预算详情 |
| `/api/cpm/budgets` | POST | BudgetController.java | 创建预算 |
| `/api/cpm/budgets/{id}/execute` | POST | BudgetController.java | 执行预算 |
| `/api/cpm/budgets/{id}/analyze` | GET | BudgetController.java | 分析预算 |

### 7.3 绩效评估接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cpm/kpis` | GET | PerformanceController.java | 查询 KPI 列表 |
| `/api/cpm/kpis` | POST | PerformanceController.java | 创建 KPI |
| `/api/cpm/kpis/{id}/evaluate` | POST | PerformanceController.java | 评估绩效 |
| `/api/cpm/performance/ranking` | GET | PerformanceController.java | 获取绩效排名 |

### 7.4 平衡计分卡接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cpm/scorecard` | GET | ScorecardController.java | 获取计分卡 |
| `/api/cpm/scorecard/design` | POST | ScorecardController.java | 设计计分卡 |
| `/api/cpm/scorecard/track` | GET | ScorecardController.java | 跟踪指标 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 战略管理 | strategy:read, strategy:write | 查看和管理战略 |
| 预算管理 | budget:read, budget:write | 查看和管理预算 |
| 绩效评估 | performance:read, performance:write | 查看和评估绩效 |

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
| BI | REST API | 绩效数据可视化 |
| FMS | REST API | 财务数据同步 |
| ERP | REST API | 业务数据同步 |
| HR | REST API | 员工绩效数据 |

---

**文档结束**
