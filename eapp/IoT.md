# IoT 物联网平台系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 IoT（Internet of Things）物联网平台系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
IoT 系统作为企业级应用体系的物联网平台，负责设备接入、数据采集、设备管理、规则引擎、数据分析等物联网业务，与 EAM、EMS、MES 系统紧密协同，实现设备互联互通和智能化管理。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 设备接入 | 设备注册、协议适配、认证管理 | 高 |
| 2 | 数据采集 | 实时数据采集、批量数据上报 | 高 |
| 3 | 设备管理 | 设备列表、设备状态、设备配置 | 高 |
| 4 | 规则引擎 | 数据规则、告警规则、自动化规则 | 高 |
| 5 | 设备控制 | 远程控制、命令下发、设备联动 | 高 |
| 6 | 数据分析 | 数据统计、趋势分析、异常检测 | 中 |
| 7 | 可视化 | 设备看板、实时监控、地理定位 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 支持百万级设备并发，消息延迟 < 100ms |
| 可用性 | 99.99% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **事件驱动**: 通过消息队列实现实时数据处理

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 接入模块 | 设备接入 | 注册、协议适配 |
| 采集模块 | 数据采集 | 实时采集、批量上报 |
| 设备模块 | 设备管理 | 列表、状态、配置 |
| 规则模块 | 规则引擎 | 规则配置、执行 |
| 控制模块 | 设备控制 | 远程控制、命令下发 |
| 分析模块 | 数据分析 | 统计、趋势、异常 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/iot/
  │   │   │   ├── controller/
  │   │   │   │   ├── DeviceController.java    # 设备管理
  │   │   │   │   ├── DataController.java      # 数据采集
  │   │   │   │   ├── RuleController.java      # 规则引擎
  │   │   │   │   └── ControlController.java   # 设备控制
  │   │   │   ├── service/
  │   │   │   │   ├── DeviceService.java
  │   │   │   │   ├── DataCollectionService.java
  │   │   │   │   ├── RuleEngineService.java
  │   │   │   │   └── ControlService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── IotApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── device/                          # 设备组件
  │   │   │   └── DeviceCard.vue
  │   ├── views/
  │   │   ├── device/                          # 设备管理
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── monitor/                         # 实时监控
  │   │   │   └── index.vue
  │   │   ├── rule/                            # 规则引擎
  │   │   │   └── index.vue
  │   │   ├── control/                         # 设备控制
  │   │   │   └── index.vue
  │   │   └── analysis/                        # 数据分析
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 DeviceService (设备服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `registerDevice` | 注册设备 | `DeviceRegisterRequest request` | `DeviceResponse` | 抛出`BusinessException` |
| `activateDevice` | 激活设备 | `String deviceId` | `DeviceResponse` | 抛出`DeviceNotFoundException` |
| `getDeviceStatus` | 获取设备状态 | `String deviceId` | `DeviceStatusResponse` | 抛出`DeviceNotFoundException` |
| `getDevices` | 查询设备列表 | `DeviceSearchRequest request` | `Page<DeviceResponse>` | - |

#### 5.1.2 DataCollectionService (数据采集服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `collectData` | 采集设备数据 | `DeviceDataRequest request` | `DeviceDataResponse` | 抛出`DeviceNotFoundException` |
| `batchCollect` | 批量采集数据 | `List<DeviceDataRequest> requests` | `List<DeviceDataResponse>` | - |
| `queryHistoryData` | 查询历史数据 | `HistoryDataRequest request` | `List<DeviceDataResponse>` | - |

#### 5.1.3 RuleEngineService (规则引擎服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createRule` | 创建规则 | `RuleCreateRequest request` | `RuleResponse` | 抛出`BusinessException` |
| `enableRule` | 启用规则 | `Long ruleId` | `RuleResponse` | 抛出`RuleNotFoundException` |
| `executeRule` | 执行规则 | `Long ruleId, RuleExecutionContext context` | `RuleExecutionResponse` | 抛出`RuleNotFoundException` |

### 5.2 DTO 结构定义

**DeviceRegisterRequest（注册设备请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| deviceId | String | 设备 ID | 非空，唯一 |
| deviceName | String | 设备名称 | 非空 |
| deviceType | String | 设备类型 | 非空 |
| protocolType | String | 协议类型 | 非空 |
| location | String | 安装位置 | 非空 |
| metadata | Map<String, String> | 设备元数据 | 可选 |

**DeviceDataRequest（设备数据请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| deviceId | String | 设备 ID | 非空 |
| timestamp | LocalDateTime | 数据时间戳 | 非空 |
| measurements | Map<String, Object> | 测量数据 | 非空 |
| status | String | 设备状态 | 可选 |

**RuleCreateRequest（创建规则请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| ruleName | String | 规则名称 | 非空 |
| ruleType | String | 规则类型 | 非空 |
| condition | String | 触发条件 | 非空 |
| actions | List<RuleAction> | 执行动作 | 非空 |
| deviceIds | List<String> | 适用设备 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 设备表 (iot_device)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | VARCHAR(50) | PRIMARY KEY | 设备 ID |
| device_name | VARCHAR(100) | NOT NULL | 设备名称 |
| device_type | VARCHAR(50) | NOT NULL | 设备类型 |
| protocol_type | VARCHAR(50) | NOT NULL | 协议类型 |
| status | VARCHAR(20) | NOT NULL | 设备状态 |
| location | VARCHAR(200) | NOT NULL | 安装位置 |
| metadata | TEXT | - | 设备元数据(JSON) |
| last_active_time | DATETIME | - | 最后活跃时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 设备数据表 (iot_device_data)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 数据 ID |
| device_id | VARCHAR(50) | NOT NULL | 设备 ID |
| timestamp | DATETIME | NOT NULL | 数据时间戳 |
| measurements | TEXT | NOT NULL | 测量数据(JSON) |
| status | VARCHAR(20) | - | 设备状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 规则表 (iot_rule)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 规则 ID |
| rule_name | VARCHAR(100) | NOT NULL | 规则名称 |
| rule_type | VARCHAR(50) | NOT NULL | 规则类型 |
| condition | TEXT | NOT NULL | 触发条件 |
| actions | TEXT | NOT NULL | 执行动作(JSON) |
| device_ids | TEXT | NOT NULL | 适用设备(JSON) |
| status | VARCHAR(20) | NOT NULL | 规则状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 设备命令表 (iot_device_command)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 命令 ID |
| device_id | VARCHAR(50) | NOT NULL | 设备 ID |
| command_type | VARCHAR(50) | NOT NULL | 命令类型 |
| command_data | TEXT | NOT NULL | 命令数据(JSON) |
| status | VARCHAR(20) | NOT NULL | 命令状态 |
| sent_at | DATETIME | - | 发送时间 |
| executed_at | DATETIME | - | 执行时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 设备管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/iot/devices` | GET | DeviceController.java | 查询设备列表 |
| `/api/iot/devices/{id}` | GET | DeviceController.java | 查询设备详情 |
| `/api/iot/devices/register` | POST | DeviceController.java | 注册设备 |
| `/api/iot/devices/{id}/activate` | POST | DeviceController.java | 激活设备 |

### 7.2 数据采集接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/iot/data/collect` | POST | DataController.java | 采集设备数据 |
| `/api/iot/data/batch` | POST | DataController.java | 批量采集数据 |
| `/api/iot/data/history` | GET | DataController.java | 查询历史数据 |

### 7.3 规则引擎接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/iot/rules` | GET | RuleController.java | 查询规则列表 |
| `/api/iot/rules` | POST | RuleController.java | 创建规则 |
| `/api/iot/rules/{id}/enable` | POST | RuleController.java | 启用规则 |

### 7.4 设备控制接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/iot/control/command` | POST | ControlController.java | 下发命令 |
| `/api/iot/control/{deviceId}` | GET | ControlController.java | 获取设备命令状态 |

---

## 8. 安全设计

### 8.1 认证机制
- 设备采用 Token/证书认证
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 设备管理 | device:read, device:write | 查看和管理设备 |
| 数据采集 | data:read, data:write | 查看和采集数据 |
| 规则管理 | rule:read, rule:write | 查看和创建规则 |

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
| InfluxDB | 2.x | 时序数据库 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| EAM | REST API + 消息队列 | 设备维护协同 |
| EMS | REST API + 消息队列 | 能源数据采集 |
| MES | REST API + 消息队列 | 生产数据采集 |

---

**文档结束**