# MES 制造执行系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 MES（Manufacturing Execution System）制造执行系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
MES 系统作为企业级应用体系的制造执行平台，负责生产计划、工序作业、质量采集、设备监控、物料追踪等制造业务，与 ERP、PLM、WMS、QMS 系统紧密协同，实现生产过程的透明化和精细化管理。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 生产计划 | 生产工单、作业排程、工序计划 | 高 |
| 2 | 工序作业 | 工序执行、报工管理、作业记录 | 高 |
| 3 | 质量采集 | 质量检验、质量追溯、SPC 分析 | 高 |
| 4 | 设备监控 | 设备状态、OEE 计算、报警管理 | 高 |
| 5 | 物料追踪 | 物料投入、成品入库、批次追溯 | 高 |
| 6 | 工艺管理 | 工艺路线、工艺参数、工艺变更 | 中 |
| 7 | 生产统计 | 产量统计、良率分析、效率分析 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 响应时间 < 100ms，实时性要求高 |
| 可用性 | 99.95% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **事件驱动**: 通过消息队列实现实时协同

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 计划模块 | 生产计划 | 工单、作业排程 |
| 工序模块 | 工序作业 | 执行、报工 |
| 质量模块 | 质量采集 | 检验、追溯 |
| 设备模块 | 设备监控 | 状态、OEE |
| 物料模块 | 物料追踪 | 投入、入库 |
| 工艺模块 | 工艺管理 | 路线、参数 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/mes/
  │   │   │   ├── controller/
  │   │   │   │   ├── ProductionController.java   # 生产计划
  │   │   │   │   ├── ProcessController.java      # 工序作业
  │   │   │   │   ├── QualityController.java      # 质量采集
  │   │   │   │   ├── EquipmentController.java    # 设备监控
  │   │   │   │   └── TraceController.java        # 物料追踪
  │   │   │   ├── service/
  │   │   │   │   ├── ProductionService.java
  │   │   │   │   ├── ProcessService.java
  │   │   │   │   ├── QualityService.java
  │   │   │   │   ├── EquipmentService.java
  │   │   │   │   └── TraceService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── MesApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── dashboard/                        # 看板组件
  │   │   │   └── ProductionDashboard.vue
  │   ├── views/
  │   │   ├── production/                      # 生产计划
  │   │   │   ├── list.vue
  │   │   │   └── schedule.vue
  │   │   ├── process/                         # 工序作业
  │   │   │   └── index.vue
  │   │   ├── quality/                         # 质量采集
  │   │   │   └── index.vue
  │   │   ├── equipment/                       # 设备监控
  │   │   │   └── index.vue
  │   │   └── trace/                           # 物料追踪
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ProductionService (生产计划服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createWorkOrder` | 创建工单 | `WorkOrderCreateRequest request` | `WorkOrderResponse` | 抛出`BusinessException` |
| `releaseWorkOrder` | 发布工单 | `Long workOrderId` | `WorkOrderResponse` | 抛出`WorkOrderNotFoundException` |
| `scheduleWorkOrders` | 排程工单 | `ScheduleRequest request` | `ScheduleResponse` | 抛出`BusinessException` |
| `getWorkOrders` | 查询工单列表 | `WorkOrderSearchRequest request` | `Page<WorkOrderResponse>` | - |

#### 5.1.2 ProcessService (工序作业服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `startProcess` | 开始工序 | `Long processId` | `ProcessResponse` | 抛出`ProcessNotFoundException` |
| `reportProcess` | 工序报工 | `Long processId, ReportRequest request` | `ReportResponse` | 抛出`ProcessNotFoundException` |
| `completeProcess` | 完工工序 | `Long processId` | `ProcessResponse` | 抛出`ProcessNotFoundException` |

#### 5.1.3 EquipmentService (设备监控服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `getEquipmentStatus` | 获取设备状态 | `String equipmentId` | `EquipmentStatusResponse` | 抛出`EquipmentNotFoundException` |
| `calculateOEE` | 计算 OEE | `String equipmentId, DateRange range` | `OEEResponse` | - |
| `getAlerts` | 获取设备报警 | `AlertSearchRequest request` | `Page<AlertResponse>` | - |

### 5.2 DTO 结构定义

**WorkOrderCreateRequest（创建工单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| workOrderNo | String | 工单编号 | 非空，唯一 |
| productId | Long | 产品 ID | 非空 |
| routeId | Long | 工艺路线 ID | 非空 |
| quantity | BigDecimal | 计划数量 | 非空 |
| plannedStartDate | LocalDateTime | 计划开始时间 | 非空 |
| plannedEndDate | LocalDateTime | 计划结束时间 | 非空 |
| priority | String | 优先级 | 非空 |

**ReportRequest（报工请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| processId | Long | 工序 ID | 非空 |
| quantity | BigDecimal | 报工数量 | 非空 |
| qualifiedQuantity | BigDecimal | 合格数量 | 非空 |
| startTime | LocalDateTime | 开始时间 | 非空 |
| endTime | LocalDateTime | 结束时间 | 非空 |
| remark | String | 备注 | 可选 |

**OEEResponse（OEE 响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| equipmentId | String | 设备 ID |
| availability | BigDecimal | 可用率 |
| performance | BigDecimal | 性能率 |
| quality | BigDecimal | 质量率 |
| oee | BigDecimal | OEE 值 |
| calculatedAt | LocalDateTime | 计算时间 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 工单表 (mes_work_order)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 工单 ID |
| work_order_no | VARCHAR(50) | UNIQUE, NOT NULL | 工单编号 |
| product_id | BIGINT | NOT NULL | 产品 ID |
| route_id | BIGINT | NOT NULL | 工艺路线 ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| planned_quantity | DECIMAL(15,4) | NOT NULL | 计划数量 |
| completed_quantity | DECIMAL(15,4) | DEFAULT 0 | 完成数量 |
| qualified_quantity | DECIMAL(15,4) | DEFAULT 0 | 合格数量 |
| planned_start_date | DATETIME | NOT NULL | 计划开始时间 |
| planned_end_date | DATETIME | NOT NULL | 计划结束时间 |
| actual_start_date | DATETIME | - | 实际开始时间 |
| actual_end_date | DATETIME | - | 实际结束时间 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 工序作业表 (mes_process_execution)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 作业 ID |
| work_order_id | BIGINT | FOREIGN KEY, NOT NULL | 工单 ID |
| process_id | BIGINT | NOT NULL | 工序 ID |
| process_name | VARCHAR(100) | NOT NULL | 工序名称 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| workstation_id | VARCHAR(50) | - | 工位 ID |
| worker_id | BIGINT | - | 作业员 ID |
| planned_start_time | DATETIME | NOT NULL | 计划开始时间 |
| planned_end_time | DATETIME | NOT NULL | 计划结束时间 |
| actual_start_time | DATETIME | - | 实际开始时间 |
| actual_end_time | DATETIME | - | 实际结束时间 |
| reported_quantity | DECIMAL(15,4) | DEFAULT 0 | 报工数量 |
| qualified_quantity | DECIMAL(15,4) | DEFAULT 0 | 合格数量 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 质量数据表 (mes_quality_data)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 数据 ID |
| work_order_id | BIGINT | NOT NULL | 工单 ID |
| process_id | BIGINT | NOT NULL | 工序 ID |
| inspection_type | VARCHAR(50) | NOT NULL | 检验类型 |
| sample_quantity | INT | NOT NULL | 抽样数量 |
| qualified_quantity | INT | NOT NULL | 合格数量 |
| defect_quantity | INT | DEFAULT 0 | 不合格数量 |
| pass_rate | DECIMAL(5,2) | NOT NULL | 通过率 |
| inspector_id | BIGINT | NOT NULL | 检验员 ID |
| inspected_at | DATETIME | NOT NULL | 检验时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 设备状态表 (mes_equipment_status)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 状态 ID |
| equipment_id | VARCHAR(50) | NOT NULL | 设备 ID |
| equipment_name | VARCHAR(100) | NOT NULL | 设备名称 |
| status | VARCHAR(20) | NOT NULL | 设备状态 |
| running_time | INT | DEFAULT 0 | 运行时间(分钟) |
| downtime | INT | DEFAULT 0 | 停机时间(分钟) |
| output_quantity | DECIMAL(15,4) | DEFAULT 0 | 产出数量 |
| efficiency | DECIMAL(5,2) | DEFAULT 0 | 效率 |
| recorded_at | DATETIME | NOT NULL | 记录时间 |

---

## 7. API 接口设计

### 7.1 生产计划接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/mes/work-orders` | GET | ProductionController.java | 查询工单列表 |
| `/api/mes/work-orders/{id}` | GET | ProductionController.java | 查询工单详情 |
| `/api/mes/work-orders` | POST | ProductionController.java | 创建工单 |
| `/api/mes/work-orders/{id}/release` | POST | ProductionController.java | 发布工单 |
| `/api/mes/work-orders/schedule` | POST | ProductionController.java | 工单排程 |

### 7.2 工序作业接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/mes/process/start` | POST | ProcessController.java | 开始工序 |
| `/api/mes/process/report` | POST | ProcessController.java | 工序报工 |
| `/api/mes/process/complete` | POST | ProcessController.java | 完工工序 |

### 7.3 质量采集接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/mes/quality/data` | GET | QualityController.java | 查询质量数据 |
| `/api/mes/quality/data` | POST | QualityController.java | 录入质量数据 |

### 7.4 设备监控接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/mes/equipment/{id}/status` | GET | EquipmentController.java | 获取设备状态 |
| `/api/mes/equipment/{id}/oee` | GET | EquipmentController.java | 计算 OEE |
| `/api/mes/equipment/alerts` | GET | EquipmentController.java | 查询设备报警 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 工单管理 | workorder:read, workorder:write | 查看和创建工单 |
| 工序作业 | process:read, process:write | 查看和执行工序 |
| 质量采集 | quality:read, quality:write | 查看和录入质量数据 |

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
| ERP | REST API + 消息队列 | 生产订单同步 |
| PLM | REST API | 工艺路线获取 |
| WMS | REST API + 消息队列 | 物料消耗同步 |
| QMS | REST API | 质量检验协同 |

---

**文档结束**