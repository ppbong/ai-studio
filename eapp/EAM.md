# EAM 设备资产管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 EAM（Enterprise Asset Management）设备资产管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
EAM 系统作为企业级应用体系的设备资产管理平台，负责设备台账、设备维护、设备维修、备件管理等设备全生命周期管理，与 ERP、WMS 系统紧密协同，确保设备高效运行。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 设备台账 | 设备信息、设备分类、设备档案 | 高 |
| 2 | 设备维护 | 预防性维护、维护计划、维护记录 | 高 |
| 3 | 设备维修 | 故障报修、维修工单、维修记录 | 高 |
| 4 | 备件管理 | 备件库存、备件领用、备件采购 | 高 |
| 5 | 设备巡检 | 巡检计划、巡检记录、巡检异常 | 高 |
| 6 | 设备报废 | 报废申请、报废审批、报废处理 | 中 |
| 7 | 设备报表 | 设备利用率、维护成本、故障率 | 中 |

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
| 设备模块 | 设备台账 | 设备信息、档案管理 |
| 维护模块 | 设备维护 | 预防性维护、维护计划 |
| 维修模块 | 设备维修 | 故障报修、维修工单 |
| 备件模块 | 备件管理 | 备件库存、领用 |
| 巡检模块 | 设备巡检 | 巡检计划、记录 |
| 报表模块 | 设备报表 | 利用率、成本统计 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/eam/
  │   │   │   ├── controller/
  │   │   │   │   ├── EquipmentController.java   # 设备管理
  │   │   │   │   ├── MaintenanceController.java # 维护管理
  │   │   │   │   ├── RepairController.java      # 维修管理
  │   │   │   │   ├── SparePartController.java   # 备件管理
  │   │   │   │   └── InspectionController.java  # 巡检管理
  │   │   │   ├── service/
  │   │   │   │   ├── EquipmentService.java
  │   │   │   │   ├── MaintenanceService.java
  │   │   │   │   ├── RepairService.java
  │   │   │   │   ├── SparePartService.java
  │   │   │   │   └── InspectionService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── EamApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   ├── views/
  │   │   ├── equipment/                         # 设备管理
  │   │   │   └── index.vue
  │   │   ├── maintenance/                       # 维护管理
  │   │   │   └── index.vue
  │   │   ├── repair/                            # 维修管理
  │   │   │   └── index.vue
  │   │   ├── sparepart/                         # 备件管理
  │   │   │   └── index.vue
  │   │   ├── inspection/                        # 巡检管理
  │   │   │   └── index.vue
  │   │   └── report/                            # 设备报表
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 EquipmentService (设备服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createEquipment` | 创建设备 | `EquipmentCreateRequest request` | `EquipmentResponse` | 抛出`BusinessException` |
| `scrapEquipment` | 设备报废 | `Long equipmentId, ScrapRequest request` | `EquipmentResponse` | 抛出`EquipmentNotFoundException` |
| `getEquipments` | 查询设备列表 | `EquipmentSearchRequest request` | `Page<EquipmentResponse>` | - |

#### 5.1.2 MaintenanceService (维护服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createMaintenancePlan` | 创建维护计划 | `MaintenancePlanCreateRequest request` | `MaintenancePlanResponse` | 抛出`BusinessException` |
| `executeMaintenance` | 执行维护 | `Long planId, MaintenanceExecuteRequest request` | `MaintenanceRecordResponse` | 抛出`PlanNotFoundException` |
| `getMaintenanceRecords` | 查询维护记录 | `MaintenanceSearchRequest request` | `Page<MaintenanceRecordResponse>` | - |

#### 5.1.3 RepairService (维修服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createRepairOrder` | 创建维修工单 | `RepairOrderCreateRequest request` | `RepairOrderResponse` | 抛出`BusinessException` |
| `assignRepair` | 分配维修 | `Long orderId, Long technicianId` | `RepairOrderResponse` | 抛出`OrderNotFoundException` |
| `completeRepair` | 完成维修 | `Long orderId, RepairCompleteRequest request` | `RepairOrderResponse` | 抛出`OrderNotFoundException` |

### 5.2 DTO 结构定义

**EquipmentCreateRequest（创建设备请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| equipmentNo | String | 设备编号 | 非空，唯一 |
| equipmentName | String | 设备名称 | 非空 |
| category | String | 设备分类 | 非空 |
| model | String | 设备型号 | 非空 |
| manufacturer | String | 制造商 | 可选 |
| purchaseDate | LocalDate | 采购日期 | 非空 |
| location | String | 安装位置 | 非空 |

**MaintenancePlanCreateRequest（创建维护计划请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| equipmentId | Long | 设备 ID | 非空 |
| planName | String | 计划名称 | 非空 |
| cycleType | String | 周期类型(DAILY/WEEKLY/MONTHLY/YEARLY) | 非空 |
| cycleValue | Integer | 周期值 | 非空 |
| maintenanceItems | List<String> | 维护项目 | 非空 |

**RepairOrderCreateRequest（创建维修工单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| equipmentId | Long | 设备 ID | 非空 |
| problemDescription | String | 故障描述 | 非空 |
| priority | String | 优先级(LOW/MEDIUM/HIGH/URGENT) | 非空 |
| reportedBy | Long | 报修人 ID | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 设备表 (eam_equipment)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 设备 ID |
| equipment_no | VARCHAR(50) | UNIQUE, NOT NULL | 设备编号 |
| equipment_name | VARCHAR(200) | NOT NULL | 设备名称 |
| category | VARCHAR(50) | NOT NULL | 设备分类 |
| model | VARCHAR(100) | NOT NULL | 设备型号 |
| manufacturer | VARCHAR(100) | - | 制造商 |
| purchase_date | DATE | NOT NULL | 采购日期 |
| location | VARCHAR(200) | NOT NULL | 安装位置 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| purchase_price | DECIMAL(15,2) | - | 采购价格 |
| scrap_date | DATE | - | 报废日期 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 设备维护计划表 (eam_maintenance_plan)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 计划 ID |
| equipment_id | BIGINT | FOREIGN KEY, NOT NULL | 设备 ID |
| plan_name | VARCHAR(200) | NOT NULL | 计划名称 |
| cycle_type | VARCHAR(20) | NOT NULL | 周期类型 |
| cycle_value | INT | NOT NULL | 周期值 |
| maintenance_items | TEXT | NOT NULL | 维护项目(JSON) |
| status | VARCHAR(20) | DEFAULT 'ACTIVE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 设备维护记录表 (eam_maintenance_record)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 记录 ID |
| plan_id | BIGINT | FOREIGN KEY | 计划 ID |
| equipment_id | BIGINT | FOREIGN KEY, NOT NULL | 设备 ID |
| maintenance_date | DATE | NOT NULL | 维护日期 |
| maintenance_items | TEXT | NOT NULL | 维护项目(JSON) |
| technician_id | BIGINT | NOT NULL | 维护人员 ID |
| remarks | VARCHAR(500) | - | 备注 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 维修工单表 (eam_repair_order)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 工单 ID |
| order_no | VARCHAR(50) | UNIQUE, NOT NULL | 工单号 |
| equipment_id | BIGINT | FOREIGN KEY, NOT NULL | 设备 ID |
| problem_description | TEXT | NOT NULL | 故障描述 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| reported_by | BIGINT | NOT NULL | 报修人 ID |
| technician_id | BIGINT | - | 维修人员 ID |
| repair_cost | DECIMAL(12,2) | DEFAULT 0 | 维修费用 |
| completed_at | DATETIME | - | 完成时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 备件表 (eam_spare_part)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 备件 ID |
| part_no | VARCHAR(50) | UNIQUE, NOT NULL | 备件编号 |
| part_name | VARCHAR(200) | NOT NULL | 备件名称 |
| category | VARCHAR(50) | NOT NULL | 备件分类 |
| model | VARCHAR(100) | NOT NULL | 备件型号 |
| unit | VARCHAR(20) | NOT NULL | 单位 |
| quantity | INT | DEFAULT 0 | 库存数量 |
| min_stock | INT | DEFAULT 0 | 最低库存 |
| max_stock | INT | DEFAULT 0 | 最高库存 |
| location | VARCHAR(200) | - | 存放位置 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.6 巡检记录表 (eam_inspection_record)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 巡检 ID |
| equipment_id | BIGINT | FOREIGN KEY, NOT NULL | 设备 ID |
| inspector_id | BIGINT | NOT NULL | 巡检人员 ID |
| inspection_date | DATE | NOT NULL | 巡检日期 |
| items | TEXT | NOT NULL | 巡检项(JSON) |
| status | VARCHAR(20) | NOT NULL | 巡检状态 |
| remarks | VARCHAR(500) | - | 备注 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 设备管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/eam/equipments` | GET | EquipmentController.java | 查询设备列表 |
| `/api/eam/equipments/{id}` | GET | EquipmentController.java | 查询设备详情 |
| `/api/eam/equipments` | POST | EquipmentController.java | 创建设备 |
| `/api/eam/equipments/{id}/scrap` | POST | EquipmentController.java | 设备报废 |

### 7.2 维护管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/eam/maintenance/plans` | GET | MaintenanceController.java | 查询维护计划 |
| `/api/eam/maintenance/plans` | POST | MaintenanceController.java | 创建维护计划 |
| `/api/eam/maintenance/records` | GET | MaintenanceController.java | 查询维护记录 |
| `/api/eam/maintenance/plans/{id}/execute` | POST | MaintenanceController.java | 执行维护 |

### 7.3 维修管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/eam/repair/orders` | GET | RepairController.java | 查询维修工单 |
| `/api/eam/repair/orders` | POST | RepairController.java | 创建维修工单 |
| `/api/eam/repair/orders/{id}/assign` | POST | RepairController.java | 分配维修 |
| `/api/eam/repair/orders/{id}/complete` | POST | RepairController.java | 完成维修 |

### 7.4 备件管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/eam/spare-parts` | GET | SparePartController.java | 查询备件列表 |
| `/api/eam/spare-parts/{id}` | GET | SparePartController.java | 查询备件详情 |
| `/api/eam/spare-parts` | POST | SparePartController.java | 创建备件 |
| `/api/eam/spare-parts/{id}/issue` | POST | SparePartController.java | 领用备件 |

### 7.5 巡检管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/eam/inspections` | GET | InspectionController.java | 查询巡检记录 |
| `/api/eam/inspections` | POST | InspectionController.java | 创建巡检记录 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 设备管理 | equipment:read, equipment:write | 查看和编辑设备 |
| 维护管理 | maintenance:read, maintenance:execute | 查看和执行维护 |
| 维修管理 | repair:read, repair:write | 查看和创建工单 |
| 备件管理 | sparepart:read, sparepart:write | 查看和管理备件 |

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
| ERP | REST API + 消息队列 | 设备采购、维护成本统计 |
| WMS | REST API | 备件库存管理 |

---

**文档结束**