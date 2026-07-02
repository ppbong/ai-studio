# QMS 质量管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 QMS（Quality Management System）质量管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
QMS 系统作为企业级应用体系的质量管理平台，负责质量检验、质量控制、质量改进、不合格品管理等质量业务，与 PLM、SCM、ERP 系统紧密协同，确保产品质量和流程合规。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 质量检验 | 来料检验、过程检验、成品检验 | 高 |
| 2 | 不合格品管理 | 不合格品记录、处理、追溯 | 高 |
| 3 | 质量控制 | 质量标准、检验计划、抽样方案 | 高 |
| 4 | 质量改进 | 问题分析、纠正措施、预防措施 | 高 |
| 5 | 质量文档 | 质量手册、程序文件、作业指导书 | 中 |
| 6 | 质量报表 | 质量统计、趋势分析、KPI 报表 | 中 |
| 7 | 供应商质量 | 供应商评分、供应商审核 | 中 |

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
| 检验模块 | 质量检验 | 来料/过程/成品检验 |
| 不合格模块 | 不合格品管理 | 不合格品记录、处理 |
| 控制模块 | 质量控制 | 检验计划、质量标准 |
| 改进模块 | 质量改进 | 问题分析、纠正措施 |
| 文档模块 | 质量文档 | 质量手册、程序文件 |
| 报表模块 | 质量报表 | 统计分析、KPI |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/qms/
  │   │   │   ├── controller/
  │   │   │   │   ├── InspectionController.java   # 检验管理
  │   │   │   │   ├── NonconformanceController.java # 不合格品管理
  │   │   │   │   ├── QualityControlController.java # 质量控制
  │   │   │   │   └── ImprovementController.java   # 质量改进
  │   │   │   ├── service/
  │   │   │   │   ├── InspectionService.java
  │   │   │   │   ├── NonconformanceService.java
  │   │   │   │   ├── QualityControlService.java
  │   │   │   │   └── ImprovementService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── QmsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   ├── views/
  │   │   ├── inspection/                        # 检验管理
  │   │   │   └── index.vue
  │   │   ├── nonconformance/                   # 不合格品管理
  │   │   │   └── index.vue
  │   │   ├── control/                           # 质量控制
  │   │   │   └── index.vue
  │   │   ├── improvement/                       # 质量改进
  │   │   │   └── index.vue
  │   │   └── report/                            # 质量报表
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 InspectionService (检验服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createInspection` | 创建检验单 | `InspectionCreateRequest request` | `InspectionResponse` | 抛出`BusinessException` |
| `executeInspection` | 执行检验 | `Long inspectionId, InspectionExecuteRequest request` | `InspectionResponse` | 抛出`InspectionNotFoundException` |
| `getInspections` | 查询检验列表 | `InspectionSearchRequest request` | `Page<InspectionResponse>` | - |

#### 5.1.2 NonconformanceService (不合格品服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createNonconformance` | 创建不合格品记录 | `NonconformanceCreateRequest request` | `NonconformanceResponse` | 抛出`BusinessException` |
| `handleNonconformance` | 处理不合格品 | `Long nonconformanceId, HandleRequest request` | `NonconformanceResponse` | 抛出`NonconformanceNotFoundException` |
| `getNonconformances` | 查询不合格品列表 | `NonconformanceSearchRequest request` | `Page<NonconformanceResponse>` | - |

#### 5.1.3 ImprovementService (改进服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createCorrectiveAction` | 创建纠正措施 | `CorrectiveActionCreateRequest request` | `CorrectiveActionResponse` | 抛出`BusinessException` |
| `completeAction` | 完成措施 | `Long actionId` | `CorrectiveActionResponse` | 抛出`ActionNotFoundException` |
| `getCorrectiveActions` | 查询纠正措施 | `ActionSearchRequest request` | `Page<CorrectiveActionResponse>` | - |

### 5.2 DTO 结构定义

**InspectionCreateRequest（创建检验单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| inspectionNo | String | 检验单号 | 非空 |
| inspectionType | String | 检验类型(INCOMING/PROCESS/FINAL) | 非空 |
| sourceNo | String | 来源单号 | 非空 |
| materialId | Long | 物料/产品 ID | 非空 |
| planId | Long | 检验计划 ID | 可选 |

**NonconformanceCreateRequest（创建不合格品请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| inspectionId | Long | 关联检验单 ID | 可选 |
| materialId | Long | 物料 ID | 非空 |
| batchNo | String | 批次号 | 非空 |
| quantity | Integer | 不合格数量 | 非空 |
| defectType | String | 缺陷类型 | 非空 |
| description | String | 描述 | 非空 |

**CorrectiveActionCreateRequest（创建纠正措施请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| nonconformanceId | Long | 关联不合格品 ID | 可选 |
| problemDescription | String | 问题描述 | 非空 |
| rootCause | String | 根本原因 | 非空 |
| actionPlan | String | 整改方案 | 非空 |
| responsibleId | Long | 责任人 ID | 非空 |
| deadline | LocalDate | 截止日期 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 检验单表 (qms_inspection)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 检验单 ID |
| inspection_no | VARCHAR(50) | UNIQUE, NOT NULL | 检验单号 |
| inspection_type | VARCHAR(20) | NOT NULL | 检验类型 |
| source_no | VARCHAR(50) | NOT NULL | 来源单号 |
| material_id | BIGINT | NOT NULL | 物料/产品 ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| result | VARCHAR(20) | - | 检验结果 |
| inspector_id | BIGINT | - | 检验员 ID |
| inspected_at | DATETIME | - | 检验时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 检验项表 (qms_inspection_item)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 检验项 ID |
| inspection_id | BIGINT | FOREIGN KEY, NOT NULL | 检验单 ID |
| item_name | VARCHAR(100) | NOT NULL | 检验项目名称 |
| standard_value | VARCHAR(200) | NOT NULL | 标准值 |
| actual_value | VARCHAR(200) | - | 实际值 |
| result | VARCHAR(20) | NOT NULL | 结果(PASS/FAIL) |
| remark | VARCHAR(200) | - | 备注 |

#### 6.1.3 不合格品记录表 (qms_nonconformance)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 不合格品 ID |
| nonconformance_no | VARCHAR(50) | UNIQUE, NOT NULL | 不合格单号 |
| inspection_id | BIGINT | FOREIGN KEY | 检验单 ID |
| material_id | BIGINT | NOT NULL | 物料 ID |
| batch_no | VARCHAR(50) | NOT NULL | 批次号 |
| quantity | INT | NOT NULL | 不合格数量 |
| defect_type | VARCHAR(50) | NOT NULL | 缺陷类型 |
| description | TEXT | NOT NULL | 描述 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| disposal_method | VARCHAR(50) | - | 处理方式 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 纠正措施表 (qms_corrective_action)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 措施 ID |
| action_no | VARCHAR(50) | UNIQUE, NOT NULL | 措施编号 |
| nonconformance_id | BIGINT | FOREIGN KEY | 不合格品 ID |
| problem_description | TEXT | NOT NULL | 问题描述 |
| root_cause | TEXT | NOT NULL | 根本原因 |
| action_plan | TEXT | NOT NULL | 整改方案 |
| responsible_id | BIGINT | NOT NULL | 责任人 ID |
| deadline | DATE | NOT NULL | 截止日期 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| completed_at | DATETIME | - | 完成时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 质量标准表 (qms_quality_standard)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 标准 ID |
| code | VARCHAR(50) | UNIQUE, NOT NULL | 标准编码 |
| name | VARCHAR(200) | NOT NULL | 标准名称 |
| type | VARCHAR(50) | NOT NULL | 标准类型 |
| content | TEXT | NOT NULL | 标准内容 |
| status | VARCHAR(20) | DEFAULT 'ACTIVE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 检验管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/qms/inspections` | GET | InspectionController.java | 查询检验单列表 |
| `/api/qms/inspections/{id}` | GET | InspectionController.java | 查询检验单详情 |
| `/api/qms/inspections` | POST | InspectionController.java | 创建检验单 |
| `/api/qms/inspections/{id}/execute` | POST | InspectionController.java | 执行检验 |

### 7.2 不合格品管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/qms/nonconformances` | GET | NonconformanceController.java | 查询不合格品列表 |
| `/api/qms/nonconformances` | POST | NonconformanceController.java | 创建不合格品记录 |
| `/api/qms/nonconformances/{id}/handle` | POST | NonconformanceController.java | 处理不合格品 |

### 7.3 质量控制接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/qms/standards` | GET | QualityControlController.java | 查询质量标准 |
| `/api/qms/standards` | POST | QualityControlController.java | 创建质量标准 |

### 7.4 质量改进接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/qms/actions` | GET | ImprovementController.java | 查询纠正措施 |
| `/api/qms/actions` | POST | ImprovementController.java | 创建纠正措施 |
| `/api/qms/actions/{id}/complete` | POST | ImprovementController.java | 完成措施 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 检验管理 | inspection:read, inspection:write | 查看和执行检验 |
| 不合格品管理 | nonconformance:read, nonconformance:handle | 查看和处理不合格品 |
| 质量改进 | action:read, action:write | 查看和创建措施 |

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
| SCM/WMS | REST API + 消息队列 | 来料检验、库存质检 |
| PLM | REST API | 产品质量标准 |
| ERP | REST API | 质量成本统计 |

---

**文档结束**