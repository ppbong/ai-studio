# API 网关系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 API Gateway（API Gateway）网关系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
API Gateway 系统作为企业级应用体系的统一入口，负责请求路由、负载均衡、认证授权、限流熔断等网关业务，与 SSO、ESP、各业务系统紧密协同，实现 API 的统一管理和安全访问。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 路由管理 | 路由配置、路径匹配、服务转发 | 高 |
| 2 | 认证授权 | 身份认证、权限验证、令牌管理 | 高 |
| 3 | 限流熔断 | 限流策略、熔断降级、负载均衡 | 高 |
| 4 | 日志监控 | 请求日志、响应日志、监控告警 | 高 |
| 5 | 协议转换 | HTTP/HTTPS、WebSocket、gRPC | 高 |
| 6 | API 文档 | Swagger 文档、API 测试、版本管理 | 中 |
| 7 | 插件扩展 | 自定义插件、插件市场、插件管理 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 响应时间 < 10ms，支持 10 万+ QPS |
| 可用性 | 99.99% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **网关模式**: 统一入口，路由转发

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 路由模块 | 路由管理 | 配置、匹配、转发 |
| 认证模块 | 认证授权 | 认证、授权、令牌 |
| 限流模块 | 限流熔断 | 限流、熔断、降级 |
| 日志模块 | 日志监控 | 日志、监控、告警 |
| 协议模块 | 协议转换 | HTTP、WebSocket、gRPC |
| 插件模块 | 插件扩展 | 自定义、市场、管理 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/apigateway/
  │   │   │   ├── controller/
  │   │   │   │   ├── RouteController.java       # 路由管理
  │   │   │   │   ├── AuthController.java        # 认证授权
  │   │   │   │   ├── RateLimitController.java   # 限流管理
  │   │   │   │   ├── PluginController.java      # 插件管理
  │   │   │   │   └── MonitorController.java     # 监控管理
  │   │   │   ├── service/
  │   │   │   │   ├── RouteService.java
  │   │   │   │   ├── AuthService.java
  │   │   │   │   ├── RateLimitService.java
  │   │   │   │   ├── PluginService.java
  │   │   │   │   └── MonitorService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   ├── filter/
  │   │   │   └── ApiGatewayApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── route/                           # 路由组件
  │   │   │   └── RouteCard.vue
  │   ├── views/
  │   │   ├── routes/                          # 路由管理
  │   │   │   └── index.vue
  │   │   ├── ratelimit/                       # 限流管理
  │   │   │   └── index.vue
  │   │   ├── plugins/                         # 插件管理
  │   │   │   └── index.vue
  │   │   └── monitor/                         # 监控管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 RouteService (路由服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createRoute` | 创建路由 | `RouteCreateRequest request` | `RouteResponse` | 抛出`BusinessException` |
| `updateRoute` | 更新路由 | `Long routeId, RouteUpdateRequest request` | `RouteResponse` | 抛出`RouteNotFoundException` |
| `deleteRoute` | 删除路由 | `Long routeId` | `void` | 抛出`RouteNotFoundException` |
| `getRoutes` | 查询路由列表 | `RouteSearchRequest request` | `Page<RouteResponse>` | - |

#### 5.1.2 AuthService (认证服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `authenticate` | 身份认证 | `AuthRequest request` | `AuthResponse` | 抛出`AuthenticationException` |
| `authorize` | 权限验证 | `AuthorizeRequest request` | `AuthorizeResponse` | 抛出`AuthorizationException` |
| `validateToken` | 验证令牌 | `String token` | `TokenValidationResponse` | 抛出`InvalidTokenException` |

#### 5.1.3 RateLimitService (限流服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createRateLimit` | 创建限流规则 | `RateLimitCreateRequest request` | `RateLimitResponse` | 抛出`BusinessException` |
| `checkRateLimit` | 检查限流 | `RateLimitCheckRequest request` | `RateLimitCheckResponse` | - |
| `getRateLimits` | 查询限流规则 | `RateLimitSearchRequest request` | `Page<RateLimitResponse>` | - |

### 5.2 DTO 结构定义

**RouteCreateRequest（创建路由请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| routeName | String | 路由名称 | 非空 |
| routePath | String | 路由路径 | 非空 |
| serviceId | String | 服务 ID | 非空 |
| servicePath | String | 服务路径 | 非空 |
| methods | List<String> | HTTP 方法 | 非空 |
| plugins | List<String> | 插件列表 | 可选 |

**AuthRequest（认证请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| token | String | 访问令牌 | 非空 |
| path | String | 请求路径 | 非空 |
| method | String | 请求方法 | 非空 |

**RateLimitCreateRequest（创建限流规则请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| ruleName | String | 规则名称 | 非空 |
| resource | String | 资源标识 | 非空 |
| limitType | String | 限流类型 | 非空 |
| limitValue | Integer | 限流值 | 非空 |
| timeWindow | Integer | 时间窗口(秒) | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 路由表 (api_gateway_route)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 路由 ID |
| route_name | VARCHAR(100) | NOT NULL | 路由名称 |
| route_path | VARCHAR(200) | UNIQUE, NOT NULL | 路由路径 |
| service_id | VARCHAR(50) | NOT NULL | 服务 ID |
| service_path | VARCHAR(200) | NOT NULL | 服务路径 |
| methods | VARCHAR(100) | NOT NULL | HTTP 方法 |
| plugins | TEXT | - | 插件列表(JSON) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 限流规则表 (api_gateway_rate_limit)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 规则 ID |
| rule_name | VARCHAR(100) | NOT NULL | 规则名称 |
| resource | VARCHAR(200) | NOT NULL | 资源标识 |
| limit_type | VARCHAR(20) | NOT NULL | 限流类型 |
| limit_value | INT | NOT NULL | 限流值 |
| time_window | INT | NOT NULL | 时间窗口(秒) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 请求日志表 (api_gateway_request_log)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 日志 ID |
| request_id | VARCHAR(50) | NOT NULL | 请求 ID |
| route_id | BIGINT | FOREIGN KEY | 路由 ID |
| method | VARCHAR(10) | NOT NULL | 请求方法 |
| path | VARCHAR(500) | NOT NULL | 请求路径 |
| client_ip | VARCHAR(50) | - | 客户端 IP |
| user_id | BIGINT | - | 用户 ID |
| status_code | INT | NOT NULL | 响应状态码 |
| response_time | INT | NOT NULL | 响应时间(ms) |
| occurred_at | DATETIME | NOT NULL | 发生时间 |

#### 6.1.4 插件表 (api_gateway_plugin)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 插件 ID |
| plugin_name | VARCHAR(100) | NOT NULL | 插件名称 |
| plugin_code | VARCHAR(50) | UNIQUE, NOT NULL | 插件编码 |
| plugin_type | VARCHAR(50) | NOT NULL | 插件类型 |
| description | VARCHAR(500) | - | 描述 |
| config_schema | TEXT | - | 配置 Schema |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 路由插件配置表 (api_gateway_route_plugin)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 配置 ID |
| route_id | BIGINT | FOREIGN KEY, NOT NULL | 路由 ID |
| plugin_id | BIGINT | FOREIGN KEY, NOT NULL | 插件 ID |
| plugin_config | TEXT | - | 插件配置(JSON) |
| priority | INT | DEFAULT 0 | 优先级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 路由管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/gateway/routes` | GET | RouteController.java | 查询路由列表 |
| `/api/gateway/routes/{id}` | GET | RouteController.java | 查询路由详情 |
| `/api/gateway/routes` | POST | RouteController.java | 创建路由 |
| `/api/gateway/routes/{id}` | PUT | RouteController.java | 更新路由 |
| `/api/gateway/routes/{id}` | DELETE | RouteController.java | 删除路由 |

### 7.2 限流管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/gateway/ratelimits` | GET | RateLimitController.java | 查询限流规则 |
| `/api/gateway/ratelimits` | POST | RateLimitController.java | 创建限流规则 |
| `/api/gateway/ratelimits/{id}` | PUT | RateLimitController.java | 更新限流规则 |
| `/api/gateway/ratelimits/{id}` | DELETE | RateLimitController.java | 删除限流规则 |

### 7.3 插件管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/gateway/plugins` | GET | PluginController.java | 查询插件列表 |
| `/api/gateway/plugins/{id}` | GET | PluginController.java | 查询插件详情 |
| `/api/gateway/plugins` | POST | PluginController.java | 注册插件 |

### 7.4 监控管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/gateway/monitor/metrics` | GET | MonitorController.java | 获取监控指标 |
| `/api/gateway/monitor/logs` | GET | MonitorController.java | 查询请求日志 |
| `/api/gateway/monitor/alerts` | GET | MonitorController.java | 查询告警信息 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 路由管理 | route:read, route:write | 查看和管理路由 |
| 限流管理 | ratelimit:read, ratelimit:write | 查看和管理限流 |
| 插件管理 | plugin:read, plugin:write | 查看和管理插件 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Cloud Gateway | 4.1.x | 网关框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| Sentinel | 1.8.x | 限流熔断 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| ESP | REST API | 服务注册发现 |
| 各业务系统 | REST API | 路由转发 |

---

**文档结束**