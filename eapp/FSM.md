# FSM 现场服务管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 FSM（Field Service Management）现场服务管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
FSM 系统作为企业级应用体系的现场服务管理平台，负责服务工单、资源调度、现场作业、服务结算等现场服务业务，与 CRM、EAM、FMS 系统紧密协同，提升现场服务效率和客户满意度。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 服务工单 | 工单创建、工单分派、工单跟踪 | 高 |
| 2 | 资源调度 | 人员调度、车辆调度、备件调度 | 高 |
| 3 | 现场作业 | 签到打卡、作业记录、现场拍照 | 高 |
| 4 | 服务结算 | 工时统计、费用结算、发票管理 | 高 |
| 5 | 服务评价 | 客户评价、服务满意度、KPI 考核 | 高 |
| 6 | 移动应用 | 移动端工单、离线作业、实时定位 | 中 |
| 7 | 服务报表 | 服务统计、效率分析、成本分析 | 中 |

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
| 工单模块 | 服务工单 | 创建、分派、跟踪 |
| 调度模块 | 资源调度 | 人员、车辆、备件 |
| 作业模块 | 现场作业 | 签到、记录、拍照 |
| 结算模块 | 服务结算 | 工时、费用、发票 |
| 评价模块 | 服务评价 | 客户评价、满意度 |
| 报表模块 | 服务报表 | 统计、分析 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/fsm/
  │   │   │   ├── controller/
  │   │   │   │   ├── WorkOrderController.java   # 服务工单
  │   │   │   │   ├── DispatchController.java    # 资源调度
  │   │   │   │   ├── FieldWorkController.java   # 现场作业
  │   │   │   │   ├── SettlementController.java  # 服务结算
  │   │   │   │   └── EvaluationController.java  # 服务评价
  │   │   │   ├── service/
  │   │   │   │   ├── WorkOrderService.java
  │   │   │   │   ├── DispatchService.java
  │   │   │   │   ├── FieldWorkService.java
  │   │   │   │   ├── SettlementService.java
  │   │   │   │   └── EvaluationService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── FsmApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── map/                              # 地图组件
  │   │   │   └── ServiceMap.vue
  │   ├── views/
  │   │   ├── workorder/                        # 服务工单
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── dispatch/                         # 资源调度
  │   │   │   └── index.vue
  │   │   ├── fieldwork/                        # 现场作业
  │   │   │   └── index.vue
  │   │   ├── settlement/                       # 服务结算
  │   │   │   └── index.vue
  │   │   └── report/                           # 服务报表
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 WorkOrderService (工单服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createWorkOrder` | 创建工单 | `WorkOrderCreateRequest request` | `WorkOrderResponse` | 抛出`BusinessException` |
| `dispatchWorkOrder` | 分派工单 | `Long workOrderId, DispatchRequest request` | `WorkOrderResponse` | 抛出`WorkOrderNotFoundException` |
| `updateWorkOrderStatus` | 更新工单状态 | `Long workOrderId, String status` | `WorkOrderResponse` | 抛出`WorkOrderNotFoundException` |
| `getWorkOrders` | 查询工单列表 | `WorkOrderSearchRequest request` | `Page<WorkOrderResponse>` | - |

#### 5.1.2 DispatchService (调度服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `scheduleResource` | 调度资源 | `ScheduleRequest request` | `ScheduleResponse` | 抛出`BusinessException` |
| `getAvailableResources` | 获取可用资源 | `ResourceSearchRequest request` | `List<ResourceResponse>` | - |
| `getResourceSchedule` | 获取资源日程 | `Long resourceId, Date date` | `ScheduleResponse` | - |

#### 5.1.3 FieldWorkService (现场作业服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `checkIn` | 签到 | `CheckInRequest request` | `CheckInResponse` | 抛出`WorkOrderNotFoundException` |
| `checkOut` | 签退 | `Long workOrderId` | `CheckOutResponse` | 抛出`WorkOrderNotFoundException` |
| `recordWork` | 记录作业 | `WorkRecordRequest request` | `WorkRecordResponse` | 抛出`WorkOrderNotFoundException` |

### 5.2 DTO 结构定义

**WorkOrderCreateRequest（创建工单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| workOrderNo | String | 工单编号 | 非空，唯一 |
| customerId | Long | 客户 ID | 非空 |
| serviceType | String | 服务类型 | 非空 |
| serviceAddress | String | 服务地址 | 非空 |
| description | String | 问题描述 | 非空 |
| priority | String | 优先级 | 非空 |
| scheduledTime | LocalDateTime | 预约时间 | 可选 |

**ScheduleRequest（调度请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| workOrderId | Long | 工单 ID | 非空 |
| technicianId | Long | 技术人员 ID | 非空 |
| vehicleId | Long | 车辆 ID | 可选 |
| sparePartIds | List<Long> | 备件 ID 列表 | 可选 |

**WorkRecordRequest（作业记录请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| workOrderId | Long | 工单 ID | 非空 |
| workType | String | 作业类型 | 非空 |
| description | String | 作业描述 | 非空 |
| duration | Integer | 作业时长(分钟) | 非空 |
| images | List<String> | 现场照片 | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 服务工单表 (fsm_work_order)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 工单 ID |
| work_order_no | VARCHAR(50) | UNIQUE, NOT NULL | 工单编号 |
| customer_id | BIGINT | NOT NULL | 客户 ID |
| service_type | VARCHAR(50) | NOT NULL | 服务类型 |
| service_address | VARCHAR(500) | NOT NULL | 服务地址 |
| description | TEXT | NOT NULL | 问题描述 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| technician_id | BIGINT | - | 技术人员 ID |
| vehicle_id | BIGINT | - | 车辆 ID |
| scheduled_time | DATETIME | - | 预约时间 |
| actual_start_time | DATETIME | - | 实际开始时间 |
| actual_end_time | DATETIME | - | 实际结束时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 技术人员表 (fsm_technician)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 人员 ID |
| technician_no | VARCHAR(50) | UNIQUE, NOT NULL | 人员编号 |
| name | VARCHAR(100) | NOT NULL | 姓名 |
| phone | VARCHAR(20) | NOT NULL | 电话 |
| skills | VARCHAR(500) | - | 技能(JSON) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 现场作业记录表 (fsm_work_record)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 记录 ID |
| work_order_id | BIGINT | FOREIGN KEY, NOT NULL | 工单 ID |
| work_type | VARCHAR(50) | NOT NULL | 作业类型 |
| description | TEXT | NOT NULL | 作业描述 |
| duration | INT | NOT NULL | 作业时长(分钟) |
| images | TEXT | - | 现场照片(JSON) |
| technician_id | BIGINT | NOT NULL | 技术人员 ID |
| recorded_at | DATETIME | NOT NULL | 记录时间 |

#### 6.1.4 服务结算表 (fsm_settlement)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 结算 ID |
| work_order_id | BIGINT | FOREIGN KEY, NOT NULL | 工单 ID |
| technician_id | BIGINT | NOT NULL | 技术人员 ID |
| labor_hours | DECIMAL(8,2) | NOT NULL | 工时(小时) |
| labor_rate | DECIMAL(10,2) | NOT NULL | 工时费率 |
| labor_amount | DECIMAL(12,2) | NOT NULL | 工时费用 |
| material_amount | DECIMAL(12,2) | DEFAULT 0 | 材料费用 |
| other_amount | DECIMAL(12,2) | DEFAULT 0 | 其他费用 |
| total_amount | DECIMAL(12,2) | NOT NULL | 总费用 |
| invoice_no | VARCHAR(50) | - | 发票编号 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| settled_at | DATETIME | - | 结算时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 服务工单接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/fsm/work-orders` | GET | WorkOrderController.java | 查询工单列表 |
| `/api/fsm/work-orders/{id}` | GET | WorkOrderController.java | 查询工单详情 |
| `/api/fsm/work-orders` | POST | WorkOrderController.java | 创建工单 |
| `/api/fsm/work-orders/{id}/dispatch` | POST | WorkOrderController.java | 分派工单 |

### 7.2 资源调度接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/fsm/dispatch/resources` | GET | DispatchController.java | 获取可用资源 |
| `/api/fsm/dispatch/schedule` | POST | DispatchController.java | 调度资源 |
| `/api/fsm/dispatch/schedule/{resourceId}` | GET | DispatchController.java | 获取资源日程 |

### 7.3 现场作业接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/fsm/fieldwork/checkin` | POST | FieldWorkController.java | 签到 |
| `/api/fsm/fieldwork/checkout` | POST | FieldWorkController.java | 签退 |
| `/api/fsm/fieldwork/record` | POST | FieldWorkController.java | 记录作业 |

### 7.4 服务结算接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/fsm/settlement/{workOrderId}` | GET | SettlementController.java | 获取结算信息 |
| `/api/fsm/settlement/calculate` | POST | SettlementController.java | 计算费用 |
| `/api/fsm/settlement/confirm` | POST | SettlementController.java | 确认结算 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 工单管理 | workorder:read, workorder:write | 查看和创建工单 |
| 调度管理 | dispatch:read, dispatch:write | 查看和调度资源 |
| 作业管理 | fieldwork:read, fieldwork:write | 查看和记录作业 |

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
| CRM | REST API + 消息队列 | 客户信息同步 |
| EAM | REST API | 设备维护协同 |
| FMS | REST API | 费用结算 |

---

**文档结束**