# WMS仓储管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述WMS（Warehouse Management System）仓储管理系统的设计方案，包括系统架构、功能模块、API接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
WMS系统作为企业级应用体系的仓储管理平台，负责仓库管理、入库管理、出库管理、库存盘点等仓储业务，与SCM系统紧密协同，实现供应链的高效运转。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 仓库管理 | 仓库信息、库区管理、货架管理 | 高 |
| 2 | 入库管理 | 采购入库、退货入库、入库质检 | 高 |
| 3 | 出库管理 | 销售出库、调拨出库、出库复核 | 高 |
| 4 | 库存管理 | 库存查询、库存预警、库存盘点 | 高 |
| 5 | 批次管理 | 批次号、效期管理、先进先出 | 高 |
| 6 | 条码管理 | 条码生成、扫码出入库 | 高 |
| 7 | 报表统计 | 库存报表、出入库报表 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 扫码响应时间 < 100ms |
| 可用性 | 99.9%高可用 |
| 安全性 | 符合等保2.0三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **事件驱动**: 通过消息队列与SCM系统协同

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 仓库模块 | 仓库管理 | 仓库、库区、货架维护 |
| 入库模块 | 入库管理 | 采购入库、退货入库 |
| 出库模块 | 出库管理 | 销售出库、调拨出库 |
| 库存模块 | 库存管理 | 库存查询、盘点 |
| 批次模块 | 批次管理 | 批次号、效期管理 |
| 条码模块 | 条码管理 | 条码生成、扫码 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/wms/
  │   │   │   ├── controller/
  │   │   │   │   ├── WarehouseController.java  # 仓库管理
  │   │   │   │   ├── InboundController.java    # 入库管理
  │   │   │   │   ├── OutboundController.java   # 出库管理
  │   │   │   │   ├── InventoryController.java  # 库存管理
  │   │   │   │   └── BarcodeController.java    # 条码管理
  │   │   │   ├── service/
  │   │   │   │   ├── WarehouseService.java
  │   │   │   │   ├── InboundService.java
  │   │   │   │   ├── OutboundService.java
  │   │   │   │   ├── InventoryService.java
  │   │   │   │   └── BarcodeService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   ├── client/
  │   │   │   │   └── ScmClient.java
  │   │   │   └── WmsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   ├── views/
  │   │   ├── warehouse/                       # 仓库管理
  │   │   │   └── index.vue
  │   │   ├── inbound/                         # 入库管理
  │   │   │   └── index.vue
  │   │   ├── outbound/                        # 出库管理
  │   │   │   └── index.vue
  │   │   ├── inventory/                       # 库存管理
  │   │   │   └── index.vue
  │   │   └── barcode/                         # 条码管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 InboundService (入库服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createInboundOrder` | 创建入库单 | `InboundOrderCreateRequest request` | `InboundOrderResponse` | 抛出`BusinessException` |
| `confirmInbound` | 确认入库 | `Long inboundOrderId` | `InboundOrderResponse` | 抛出`OrderNotFoundException` |
| `scanInbound` | 扫码入库 | `String barcode, Integer quantity` | `ScanResult` | 抛出`ScanException` |

#### 5.1.2 OutboundService (出库服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createOutboundOrder` | 创建出库单 | `OutboundOrderCreateRequest request` | `OutboundOrderResponse` | 抛出`BusinessException` |
| `pickGoods` | 拣货 | `Long outboundOrderId, List<PickItem> items` | `PickResult` | 抛出`OrderNotFoundException` |
| `confirmOutbound` | 确认出库 | `Long outboundOrderId` | `OutboundOrderResponse` | 抛出`OrderNotFoundException` |

#### 5.1.3 InventoryService (库存服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `getInventory` | 查询库存 | `InventorySearchRequest request` | `Page<InventoryResponse>` | - |
| `inventoryCheck` | 库存盘点 | `Long warehouseId, List<CheckItem> items` | `CheckResult` | 抛出`WarehouseNotFoundException` |
| `adjustStock` | 库存调整 | `Long inventoryId, Integer quantity, String reason` | `InventoryResponse` | 抛出`InventoryNotFoundException` |

### 5.2 DTO结构定义

**InboundOrderCreateRequest（创建入库单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| orderNo | String | 入库单号 | 非空 |
| warehouseId | Long | 仓库ID | 非空 |
| sourceType | String | 来源类型(PURCHASE/RETURN) | 非空 |
| sourceNo | String | 来源单号 | 非空 |
| items | List<InboundItemRequest> | 入库商品列表 | 非空 |

**OutboundOrderCreateRequest（创建出库单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| orderNo | String | 出库单号 | 非空 |
| warehouseId | Long | 仓库ID | 非空 |
| sourceType | String | 来源类型(SALE/TRANSFER) | 非空 |
| sourceNo | String | 来源单号 | 非空 |
| items | List<OutboundItemRequest> | 出库商品列表 | 非空 |

**InventoryResponse（库存响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| id | Long | 库存ID |
| warehouseId | Long | 仓库ID |
| warehouseName | String | 仓库名称 |
| materialId | Long | 物料ID |
| materialCode | String | 物料编码 |
| location | String | 库位 |
| batchNo | String | 批次号 |
| quantity | Integer | 数量 |
| expiryDate | LocalDate | 效期 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 仓库表 (wms_warehouse)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 仓库ID |
| code | VARCHAR(50) | UNIQUE, NOT NULL | 仓库编码 |
| name | VARCHAR(100) | NOT NULL | 仓库名称 |
| address | VARCHAR(500) | - | 仓库地址 |
| status | VARCHAR(20) | DEFAULT 'ACTIVE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 库位表 (wms_location)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 库位ID |
| warehouse_id | BIGINT | FOREIGN KEY, NOT NULL | 仓库ID |
| code | VARCHAR(50) | UNIQUE, NOT NULL | 库位编码 |
| zone | VARCHAR(50) | NOT NULL | 库区 |
| row | VARCHAR(20) | NOT NULL | 排 |
| shelf | VARCHAR(20) | NOT NULL | 架 |
| level | VARCHAR(20) | NOT NULL | 层 |
| status | VARCHAR(20) | DEFAULT 'AVAILABLE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 入库单表 (wms_inbound_order)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 入库单ID |
| order_no | VARCHAR(50) | UNIQUE, NOT NULL | 入库单号 |
| warehouse_id | BIGINT | FOREIGN KEY, NOT NULL | 仓库ID |
| source_type | VARCHAR(20) | NOT NULL | 来源类型 |
| source_no | VARCHAR(50) | NOT NULL | 来源单号 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 出库单表 (wms_outbound_order)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 出库单ID |
| order_no | VARCHAR(50) | UNIQUE, NOT NULL | 出库单号 |
| warehouse_id | BIGINT | FOREIGN KEY, NOT NULL | 仓库ID |
| source_type | VARCHAR(20) | NOT NULL | 来源类型 |
| source_no | VARCHAR(50) | NOT NULL | 来源单号 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 库存表 (wms_inventory)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 库存ID |
| warehouse_id | BIGINT | FOREIGN KEY, NOT NULL | 仓库ID |
| location_id | BIGINT | FOREIGN KEY, NOT NULL | 库位ID |
| material_id | BIGINT | NOT NULL | 物料ID |
| material_code | VARCHAR(50) | NOT NULL | 物料编码 |
| batch_no | VARCHAR(50) | - | 批次号 |
| quantity | INT | NOT NULL | 数量 |
| expiry_date | DATE | - | 效期 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

---

## 7. API接口设计

### 7.1 仓库管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/wms/warehouses` | GET | WarehouseController.java | 查询仓库列表 |
| `/api/wms/warehouses/{id}` | GET | WarehouseController.java | 查询仓库详情 |
| `/api/wms/warehouses` | POST | WarehouseController.java | 创建仓库 |

### 7.2 入库管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/wms/inbound/orders` | GET | InboundController.java | 查询入库单列表 |
| `/api/wms/inbound/orders` | POST | InboundController.java | 创建入库单 |
| `/api/wms/inbound/orders/{id}/confirm` | POST | InboundController.java | 确认入库 |
| `/api/wms/inbound/scan` | POST | InboundController.java | 扫码入库 |

### 7.3 出库管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/wms/outbound/orders` | GET | OutboundController.java | 查询出库单列表 |
| `/api/wms/outbound/orders` | POST | OutboundController.java | 创建出库单 |
| `/api/wms/outbound/orders/{id}/pick` | POST | OutboundController.java | 拣货 |
| `/api/wms/outbound/orders/{id}/confirm` | POST | OutboundController.java | 确认出库 |

### 7.4 库存管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/wms/inventory` | GET | InventoryController.java | 查询库存 |
| `/api/wms/inventory/check` | POST | InventoryController.java | 库存盘点 |
| `/api/wms/inventory/adjust` | POST | InventoryController.java | 库存调整 |

### 7.5 条码管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/wms/barcode/generate` | POST | BarcodeController.java | 生成条码 |
| `/api/wms/barcode/query` | GET | BarcodeController.java | 查询条码信息 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过SSO系统进行统一身份认证
- 使用JWT令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 仓库管理 | warehouse:read, warehouse:write | 查看和编辑仓库 |
| 入库管理 | inbound:read, inbound:write | 查看和创建入库单 |
| 出库管理 | outbound:read, outbound:write | 查看和创建出库单 |
| 库存管理 | inventory:read, inventory:write | 查看和调整库存 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| RabbitMQ | 3.12+ | 消息队列 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| SCM | REST API + 消息队列 | 采购入库、销售出库协同 |

---

**文档结束**