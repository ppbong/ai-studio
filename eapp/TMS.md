# TMS 运输管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 TMS（Transportation Management System）运输管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
TMS 系统作为企业级应用体系的运输管理平台，负责运输计划、运单管理、车辆调度、司机管理、运费结算等运输业务，与 WMS、SCM 系统紧密协同，确保货物高效运输。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 运输计划 | 运输需求、路线规划、运力调度 | 高 |
| 2 | 运单管理 | 运单创建、运单跟踪、运单状态 | 高 |
| 3 | 车辆管理 | 车辆信息、车辆调度、车辆定位 | 高 |
| 4 | 司机管理 | 司机信息、司机排班、司机考核 | 高 |
| 5 | 路线管理 | 路线规划、路线优化、费用计算 | 高 |
| 6 | 运费管理 | 运费计算、运费结算、对账管理 | 高 |
| 7 | 运输报表 | 运输统计、成本分析、KPI 报表 | 中 |

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
| 计划模块 | 运输计划 | 需求汇总、路线规划 |
| 运单模块 | 运单管理 | 运单创建、跟踪 |
| 车辆模块 | 车辆管理 | 车辆信息、调度 |
| 司机模块 | 司机管理 | 司机信息、排班 |
| 路线模块 | 路线管理 | 路线规划、优化 |
| 运费模块 | 运费管理 | 计算、结算 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/tms/
  │   │   │   ├── controller/
  │   │   │   │   ├── ShipmentController.java    # 运单管理
  │   │   │   │   ├── VehicleController.java     # 车辆管理
  │   │   │   │   ├── DriverController.java      # 司机管理
  │   │   │   │   ├── RouteController.java       # 路线管理
  │   │   │   │   └── FreightController.java     # 运费管理
  │   │   │   ├── service/
  │   │   │   │   ├── ShipmentService.java
  │   │   │   │   ├── VehicleService.java
  │   │   │   │   ├── DriverService.java
  │   │   │   │   ├── RouteService.java
  │   │   │   │   └── FreightService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── TmsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── map/                              # 地图组件
  │   │   │   └── RouteMap.vue
  │   ├── views/
  │   │   ├── shipment/                         # 运单管理
  │   │   │   ├── list.vue
  │   │   │   └── track.vue
  │   │   ├── vehicle/                          # 车辆管理
  │   │   │   └── index.vue
  │   │   ├── driver/                           # 司机管理
  │   │   │   └── index.vue
  │   │   ├── route/                            # 路线管理
  │   │   │   └── index.vue
  │   │   └── freight/                          # 运费管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ShipmentService (运单服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createShipment` | 创建运单 | `ShipmentCreateRequest request` | `ShipmentResponse` | 抛出`BusinessException` |
| `dispatchShipment` | 调度运单 | `Long shipmentId, DispatchRequest request` | `ShipmentResponse` | 抛出`ShipmentNotFoundException` |
| `updateShipmentStatus` | 更新运单状态 | `Long shipmentId, String status` | `ShipmentResponse` | 抛出`ShipmentNotFoundException` |
| `getShipments` | 查询运单列表 | `ShipmentSearchRequest request` | `Page<ShipmentResponse>` | - |

#### 5.1.2 VehicleService (车辆服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createVehicle` | 创建车辆 | `VehicleCreateRequest request` | `VehicleResponse` | 抛出`BusinessException` |
| `dispatchVehicle` | 调度车辆 | `Long vehicleId, Long shipmentId` | `VehicleResponse` | 抛出`VehicleNotFoundException` |
| `getVehicleLocation` | 获取车辆位置 | `Long vehicleId` | `LocationResponse` | 抛出`VehicleNotFoundException` |
| `getVehicles` | 查询车辆列表 | `VehicleSearchRequest request` | `Page<VehicleResponse>` | - |

#### 5.1.3 RouteService (路线服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createRoute` | 创建路线 | `RouteCreateRequest request` | `RouteResponse` | 抛出`BusinessException` |
| `optimizeRoute` | 优化路线 | `Long routeId` | `RouteResponse` | 抛出`RouteNotFoundException` |
| `calculateDistance` | 计算距离 | `String origin, String destination` | `DistanceResponse` | - |

### 5.2 DTO 结构定义

**ShipmentCreateRequest（创建运单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| shipmentNo | String | 运单号 | 非空，唯一 |
| orderId | Long | 关联订单 ID | 可选 |
| customerId | Long | 客户 ID | 非空 |
| originAddress | String | 发货地址 | 非空 |
| destinationAddress | String | 收货地址 | 非空 |
| plannedPickupTime | LocalDateTime | 计划提货时间 | 非空 |
| plannedDeliveryTime | LocalDateTime | 计划送达时间 | 非空 |

**VehicleCreateRequest（创建车辆请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| vehicleNo | String | 车牌号 | 非空，唯一 |
| vehicleType | String | 车辆类型 | 非空 |
| loadCapacity | BigDecimal | 载重(吨) | 非空 |
| volumeCapacity | BigDecimal | 容积(立方米) | 非空 |
| driverId | Long | 司机 ID | 可选 |

**RouteCreateRequest（创建路线请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| routeName | String | 路线名称 | 非空 |
| origin | String | 起点 | 非空 |
| destination | String | 终点 | 非空 |
| waypoints | List<String> | 途经点 | 可选 |
| distance | BigDecimal | 距离(公里) | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 运单表 (tms_shipment)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 运单 ID |
| shipment_no | VARCHAR(50) | UNIQUE, NOT NULL | 运单号 |
| order_id | BIGINT | - | 关联订单 ID |
| customer_id | BIGINT | NOT NULL | 客户 ID |
| origin_address | VARCHAR(500) | NOT NULL | 发货地址 |
| destination_address | VARCHAR(500) | NOT NULL | 收货地址 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| vehicle_id | BIGINT | - | 车辆 ID |
| driver_id | BIGINT | - | 司机 ID |
| route_id | BIGINT | - | 路线 ID |
| planned_pickup_time | DATETIME | NOT NULL | 计划提货时间 |
| planned_delivery_time | DATETIME | NOT NULL | 计划送达时间 |
| actual_pickup_time | DATETIME | - | 实际提货时间 |
| actual_delivery_time | DATETIME | - | 实际送达时间 |
| freight_amount | DECIMAL(12,2) | - | 运费金额 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 车辆表 (tms_vehicle)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 车辆 ID |
| vehicle_no | VARCHAR(20) | UNIQUE, NOT NULL | 车牌号 |
| vehicle_type | VARCHAR(50) | NOT NULL | 车辆类型 |
| load_capacity | DECIMAL(8,2) | NOT NULL | 载重(吨) |
| volume_capacity | DECIMAL(8,2) | NOT NULL | 容积(立方米) |
| driver_id | BIGINT | - | 司机 ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| current_location | VARCHAR(200) | - | 当前位置 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 司机表 (tms_driver)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 司机 ID |
| driver_no | VARCHAR(50) | UNIQUE, NOT NULL | 司机工号 |
| name | VARCHAR(100) | NOT NULL | 姓名 |
| phone | VARCHAR(20) | NOT NULL | 电话 |
| license_no | VARCHAR(50) | NOT NULL | 驾照号 |
| license_type | VARCHAR(20) | NOT NULL | 驾照类型 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 路线表 (tms_route)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 路线 ID |
| route_name | VARCHAR(200) | NOT NULL | 路线名称 |
| origin | VARCHAR(200) | NOT NULL | 起点 |
| destination | VARCHAR(200) | NOT NULL | 终点 |
| waypoints | TEXT | - | 途经点(JSON) |
| distance | DECIMAL(10,2) | NOT NULL | 距离(公里) |
| estimated_time | INT | - | 预计时长(分钟) |
| status | VARCHAR(20) | DEFAULT 'ACTIVE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 运费结算表 (tms_freight_settlement)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 结算 ID |
| shipment_id | BIGINT | FOREIGN KEY, NOT NULL | 运单 ID |
| carrier_id | BIGINT | NOT NULL | 承运商 ID |
| freight_amount | DECIMAL(12,2) | NOT NULL | 运费金额 |
| fuel_cost | DECIMAL(12,2) | DEFAULT 0 | 油费 |
| toll_cost | DECIMAL(12,2) | DEFAULT 0 | 过路费 |
| other_cost | DECIMAL(12,2) | DEFAULT 0 | 其他费用 |
| total_cost | DECIMAL(12,2) | NOT NULL | 总费用 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| settled_at | DATETIME | - | 结算时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 运单管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/tms/shipments` | GET | ShipmentController.java | 查询运单列表 |
| `/api/tms/shipments/{id}` | GET | ShipmentController.java | 查询运单详情 |
| `/api/tms/shipments` | POST | ShipmentController.java | 创建运单 |
| `/api/tms/shipments/{id}/dispatch` | POST | ShipmentController.java | 调度运单 |
| `/api/tms/shipments/{id}/track` | GET | ShipmentController.java | 运单跟踪 |

### 7.2 车辆管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/tms/vehicles` | GET | VehicleController.java | 查询车辆列表 |
| `/api/tms/vehicles/{id}` | GET | VehicleController.java | 查询车辆详情 |
| `/api/tms/vehicles` | POST | VehicleController.java | 创建车辆 |
| `/api/tms/vehicles/{id}/location` | GET | VehicleController.java | 获取车辆位置 |

### 7.3 司机管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/tms/drivers` | GET | DriverController.java | 查询司机列表 |
| `/api/tms/drivers` | POST | DriverController.java | 创建司机 |

### 7.4 路线管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/tms/routes` | GET | RouteController.java | 查询路线列表 |
| `/api/tms/routes` | POST | RouteController.java | 创建路线 |
| `/api/tms/routes/{id}/optimize` | POST | RouteController.java | 优化路线 |

### 7.5 运费管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/tms/freight/calculate` | POST | FreightController.java | 计算运费 |
| `/api/tms/freight/settle` | POST | FreightController.java | 运费结算 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 运单管理 | shipment:read, shipment:write | 查看和创建运单 |
| 车辆管理 | vehicle:read, vehicle:write | 查看和管理车辆 |
| 司机管理 | driver:read, driver:write | 查看和管理司机 |

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
| WMS | REST API + 消息队列 | 出库单→运单 |
| SCM | REST API | 采购运输协同 |
| FMS | REST API | 运费结算 |

---

**文档结束**