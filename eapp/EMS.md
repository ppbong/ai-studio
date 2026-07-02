# EMS 能源管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 EMS（Energy Management System）能源管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
EMS 系统作为企业级应用体系的能源管理平台，负责能源数据采集、能源监控、能源分析、节能管理等能源业务，与 EAM、ERP 系统紧密协同，实现能源精细化管理和节能减排目标。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 数据采集 | 能耗数据采集、计量设备管理 | 高 |
| 2 | 能源监控 | 实时监控、异常告警、趋势分析 | 高 |
| 3 | 能源分析 | 能耗统计、成本分析、能效评估 | 高 |
| 4 | 节能管理 | 节能计划、节能项目、效果评估 | 高 |
| 5 | 设备管理 | 能源设备监控、维护管理 | 中 |
| 6 | 报表管理 | 能源报表、统计分析、KPI 报表 | 中 |
| 7 | 碳排放管理 | 碳排放计算、碳足迹跟踪 | 中 |

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
| 采集模块 | 数据采集 | 能耗数据采集、计量设备 |
| 监控模块 | 能源监控 | 实时监控、告警 |
| 分析模块 | 能源分析 | 能耗统计、成本分析 |
| 节能模块 | 节能管理 | 节能计划、项目 |
| 设备模块 | 设备管理 | 能源设备、维护 |
| 报表模块 | 报表管理 | 能源报表 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/ems/
  │   │   │   ├── controller/
  │   │   │   │   ├── DataCollectionController.java # 数据采集
  │   │   │   │   ├── MonitorController.java        # 能源监控
  │   │   │   │   ├── AnalysisController.java       # 能源分析
  │   │   │   │   └── EnergySavingController.java   # 节能管理
  │   │   │   ├── service/
  │   │   │   │   ├── DataCollectionService.java
  │   │   │   │   ├── MonitorService.java
  │   │   │   │   ├── AnalysisService.java
  │   │   │   │   └── EnergySavingService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── EmsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── charts/                           # 图表组件
  │   │   │   ├── EnergyChart.vue
  │   │   │   └── RealTimeMonitor.vue
  │   ├── views/
  │   │   ├── monitor/                          # 能源监控
  │   │   │   └── index.vue
  │   │   ├── analysis/                         # 能源分析
  │   │   │   └── index.vue
  │   │   ├── saving/                           # 节能管理
  │   │   │   └── index.vue
  │   │   ├── device/                           # 设备管理
  │   │   │   └── index.vue
  │   │   └── report/                           # 能源报表
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 DataCollectionService (数据采集服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `collectEnergyData` | 采集能源数据 | `EnergyDataRequest request` | `EnergyDataResponse` | 抛出`BusinessException` |
| `getMeterDevices` | 查询计量设备 | `MeterDeviceSearchRequest request` | `Page<MeterDeviceResponse>` | - |

#### 5.1.2 MonitorService (监控服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `getRealTimeData` | 获取实时数据 | `String deviceId` | `RealTimeDataResponse` | 抛出`DeviceNotFoundException` |
| `getAlerts` | 查询告警列表 | `AlertSearchRequest request` | `Page<AlertResponse>` | - |
| `acknowledgeAlert` | 确认告警 | `Long alertId` | `AlertResponse` | 抛出`AlertNotFoundException` |

#### 5.1.3 AnalysisService (分析服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `getEnergyStatistics` | 获取能耗统计 | `StatisticsRequest request` | `EnergyStatisticsResponse` | - |
| `getEnergyCostAnalysis` | 获取成本分析 | `CostAnalysisRequest request` | `CostAnalysisResponse` | - |
| `getEnergyEfficiency` | 获取能效评估 | `EfficiencyRequest request` | `EfficiencyResponse` | - |

### 5.2 DTO 结构定义

**EnergyDataRequest（能源数据请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| deviceId | String | 设备 ID | 非空 |
| energyType | String | 能源类型(ELECTRICITY/WATER/GAS) | 非空 |
| value | BigDecimal | 能耗值 | 非空 |
| unit | String | 单位 | 非空 |
| timestamp | LocalDateTime | 采集时间 | 非空 |

**MeterDeviceResponse（计量设备响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| id | String | 设备 ID |
| deviceCode | String | 设备编码 |
| deviceName | String | 设备名称 |
| energyType | String | 能源类型 |
| location | String | 安装位置 |
| status | String | 状态 |
| lastCollectionTime | LocalDateTime | 最后采集时间 |

**AlertResponse（告警响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| id | Long | 告警 ID |
| deviceId | String | 设备 ID |
| deviceName | String | 设备名称 |
| alertType | String | 告警类型 |
| alertLevel | String | 告警级别 |
| message | String | 告警消息 |
| status | String | 告警状态 |
| createdAt | LocalDateTime | 告警时间 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 能源数据表 (ems_energy_data)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 数据 ID |
| device_id | VARCHAR(50) | NOT NULL | 设备 ID |
| energy_type | VARCHAR(20) | NOT NULL | 能源类型 |
| value | DECIMAL(15,4) | NOT NULL | 能耗值 |
| unit | VARCHAR(20) | NOT NULL | 单位 |
| timestamp | DATETIME | NOT NULL | 采集时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 计量设备表 (ems_meter_device)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | VARCHAR(50) | PRIMARY KEY | 设备 ID |
| device_code | VARCHAR(50) | UNIQUE, NOT NULL | 设备编码 |
| device_name | VARCHAR(100) | NOT NULL | 设备名称 |
| energy_type | VARCHAR(20) | NOT NULL | 能源类型 |
| location | VARCHAR(200) | NOT NULL | 安装位置 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| last_collection_time | DATETIME | - | 最后采集时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 告警表 (ems_alert)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 告警 ID |
| device_id | VARCHAR(50) | NOT NULL | 设备 ID |
| alert_type | VARCHAR(50) | NOT NULL | 告警类型 |
| alert_level | VARCHAR(20) | NOT NULL | 告警级别 |
| message | VARCHAR(500) | NOT NULL | 告警消息 |
| status | VARCHAR(20) | NOT NULL | 告警状态 |
| acknowledged_at | DATETIME | - | 确认时间 |
| created_at | DATETIME | NOT NULL | 告警时间 |

#### 6.1.4 节能项目表 (ems_energy_saving_project)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 项目 ID |
| project_no | VARCHAR(50) | UNIQUE, NOT NULL | 项目编号 |
| project_name | VARCHAR(200) | NOT NULL | 项目名称 |
| description | TEXT | - | 项目描述 |
| expected_saving | DECIMAL(12,2) | NOT NULL | 预计节能量 |
| investment | DECIMAL(15,2) | NOT NULL | 投资金额 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| start_date | DATE | NOT NULL | 开始日期 |
| end_date | DATE | - | 结束日期 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 数据采集接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ems/data/collect` | POST | DataCollectionController.java | 采集能源数据 |
| `/api/ems/devices` | GET | DataCollectionController.java | 查询计量设备 |

### 7.2 能源监控接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ems/monitor/realtime/{deviceId}` | GET | MonitorController.java | 获取实时数据 |
| `/api/ems/monitor/alerts` | GET | MonitorController.java | 查询告警列表 |
| `/api/ems/monitor/alerts/{id}/acknowledge` | POST | MonitorController.java | 确认告警 |

### 7.3 能源分析接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ems/analysis/statistics` | GET | AnalysisController.java | 获取能耗统计 |
| `/api/ems/analysis/cost` | GET | AnalysisController.java | 获取成本分析 |
| `/api/ems/analysis/efficiency` | GET | AnalysisController.java | 获取能效评估 |

### 7.4 节能管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ems/saving/projects` | GET | EnergySavingController.java | 查询节能项目 |
| `/api/ems/saving/projects` | POST | EnergySavingController.java | 创建节能项目 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 数据采集 | data:read, data:write | 查看和采集数据 |
| 监控管理 | monitor:read, monitor:acknowledge | 查看和确认告警 |
| 节能管理 | saving:read, saving:write | 查看和创建节能项目 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| InfluxDB | 2.x | 时序数据库 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| EAM | REST API + 消息队列 | 设备维护协同 |
| ERP | REST API | 能源成本统计 |

---

**文档结束**