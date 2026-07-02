# DevOps 平台系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 DevOps 平台系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
DevOps 系统作为企业级应用体系的持续集成与持续部署平台，负责代码管理、构建部署、测试运维、监控告警等 DevOps 全流程管理，与 ESP、各业务系统紧密协同，实现应用的快速迭代和稳定运行。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 代码管理 | 代码仓库、版本控制、代码审查 | 高 |
| 2 | 持续集成 | 自动化构建、代码扫描、测试执行 | 高 |
| 3 | 持续部署 | 自动化部署、环境管理、发布流程 | 高 |
| 4 | 测试管理 | 测试用例、测试执行、测试报告 | 高 |
| 5 | 运维监控 | 应用监控、日志管理、告警通知 | 高 |
| 6 | 环境管理 | 环境配置、资源管理、权限控制 | 中 |
| 7 | 发布管理 | 发布流程、回滚机制、发布审计 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 构建任务响应 < 1min，支持多并发 |
| 可用性 | 99.9% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **CI/CD 流水线**: 自动化构建部署

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 代码模块 | 代码管理 | 仓库、版本、审查 |
| 集成模块 | 持续集成 | 构建、扫描、测试 |
| 部署模块 | 持续部署 | 部署、环境、发布 |
| 测试模块 | 测试管理 | 用例、执行、报告 |
| 运维模块 | 运维监控 | 监控、日志、告警 |
| 环境模块 | 环境管理 | 配置、资源、权限 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/devops/
  │   │   │   ├── controller/
  │   │   │   │   ├── CodeController.java       # 代码管理
  │   │   │   │   ├── CIController.java         # 持续集成
  │   │   │   │   ├── CDController.java         # 持续部署
  │   │   │   │   ├── TestController.java       # 测试管理
  │   │   │   │   └── OpsController.java        # 运维监控
  │   │   │   ├── service/
  │   │   │   │   ├── CodeService.java
  │   │   │   │   ├── CIService.java
  │   │   │   │   ├── CDService.java
  │   │   │   │   ├── TestService.java
  │   │   │   │   └── OpsService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── DevOpsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── pipeline/                        # 流水线组件
  │   │   │   └── PipelineCard.vue
  │   ├── views/
  │   │   ├── code/                            # 代码管理
  │   │   │   └── index.vue
  │   │   ├── ci/                              # 持续集成
  │   │   │   └── index.vue
  │   │   ├── cd/                              # 持续部署
  │   │   │   └── index.vue
  │   │   ├── test/                            # 测试管理
  │   │   │   └── index.vue
  │   │   └── ops/                             # 运维监控
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 CIService (持续集成服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createPipeline` | 创建流水线 | `PipelineCreateRequest request` | `PipelineResponse` | 抛出`BusinessException` |
| `runPipeline` | 执行流水线 | `Long pipelineId` | `PipelineExecutionResponse` | 抛出`PipelineNotFoundException` |
| `getPipelineStatus` | 获取流水线状态 | `Long executionId` | `PipelineStatusResponse` | 抛出`ExecutionNotFoundException` |
| `getPipelines` | 查询流水线列表 | `PipelineSearchRequest request` | `Page<PipelineResponse>` | - |

#### 5.1.2 CDService (持续部署服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createDeployment` | 创建部署 | `DeploymentCreateRequest request` | `DeploymentResponse` | 抛出`BusinessException` |
| `deploy` | 执行部署 | `Long deploymentId` | `DeploymentResponse` | 抛出`DeploymentNotFoundException` |
| `rollback` | 回滚部署 | `Long deploymentId` | `DeploymentResponse` | 抛出`DeploymentNotFoundException` |
| `getDeployments` | 查询部署列表 | `DeploymentSearchRequest request` | `Page<DeploymentResponse>` | - |

#### 5.1.3 OpsService (运维服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `getMetrics` | 获取监控指标 | `MetricsRequest request` | `MetricsResponse` | - |
| `getLogs` | 获取日志 | `LogRequest request` | `LogResponse` | - |
| `createAlert` | 创建告警规则 | `AlertCreateRequest request` | `AlertResponse` | 抛出`BusinessException` |
| `getAlerts` | 查询告警列表 | `AlertSearchRequest request` | `Page<AlertResponse>` | - |

### 5.2 DTO 结构定义

**PipelineCreateRequest（创建流水线请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| pipelineName | String | 流水线名称 | 非空 |
| repositoryUrl | String | 仓库地址 | 非空 |
| branch | String | 分支名称 | 非空 |
| stages | List<PipelineStage> | 流水线阶段 | 非空 |

**DeploymentCreateRequest（创建部署请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| deploymentName | String | 部署名称 | 非空 |
| applicationId | String | 应用 ID | 非空 |
| environment | String | 目标环境 | 非空 |
| version | String | 版本号 | 非空 |

**AlertCreateRequest（创建告警规则请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| alertName | String | 告警名称 | 非空 |
| metricType | String | 指标类型 | 非空 |
| threshold | Double | 阈值 | 非空 |
| operator | String | 比较运算符 | 非空 |
| notificationTargets | List<String> | 通知目标 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 流水线表 (devops_pipeline)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 流水线 ID |
| pipeline_name | VARCHAR(100) | NOT NULL | 流水线名称 |
| repository_url | VARCHAR(500) | NOT NULL | 仓库地址 |
| branch | VARCHAR(50) | NOT NULL | 分支名称 |
| stages | TEXT | NOT NULL | 流水线阶段(JSON) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 流水线执行表 (devops_pipeline_execution)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 执行 ID |
| pipeline_id | BIGINT | FOREIGN KEY, NOT NULL | 流水线 ID |
| commit_hash | VARCHAR(50) | - | 提交哈希 |
| status | VARCHAR(20) | NOT NULL | 执行状态 |
| start_time | DATETIME | NOT NULL | 开始时间 |
| end_time | DATETIME | - | 结束时间 |
| duration | INT | - | 执行时长(秒) |
| error_message | TEXT | - | 错误信息 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 部署表 (devops_deployment)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 部署 ID |
| deployment_name | VARCHAR(100) | NOT NULL | 部署名称 |
| application_id | VARCHAR(50) | NOT NULL | 应用 ID |
| environment | VARCHAR(50) | NOT NULL | 目标环境 |
| version | VARCHAR(20) | NOT NULL | 版本号 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| deployed_by | BIGINT | NOT NULL | 部署人 ID |
| deployed_at | DATETIME | NOT NULL | 部署时间 |

#### 6.1.4 告警规则表 (devops_alert_rule)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 规则 ID |
| alert_name | VARCHAR(100) | NOT NULL | 告警名称 |
| metric_type | VARCHAR(50) | NOT NULL | 指标类型 |
| threshold | DECIMAL(10,2) | NOT NULL | 阈值 |
| operator | VARCHAR(10) | NOT NULL | 比较运算符 |
| notification_targets | TEXT | NOT NULL | 通知目标(JSON) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 告警记录表 (devops_alert_record)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 记录 ID |
| rule_id | BIGINT | FOREIGN KEY, NOT NULL | 规则 ID |
| alert_name | VARCHAR(100) | NOT NULL | 告警名称 |
| metric_value | DECIMAL(10,2) | NOT NULL | 指标值 |
| threshold | DECIMAL(10,2) | NOT NULL | 阈值 |
| message | VARCHAR(500) | NOT NULL | 告警消息 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| occurred_at | DATETIME | NOT NULL | 发生时间 |

---

## 7. API 接口设计

### 7.1 持续集成接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/devops/ci/pipelines` | GET | CIController.java | 查询流水线列表 |
| `/api/devops/ci/pipelines` | POST | CIController.java | 创建流水线 |
| `/api/devops/ci/pipelines/{id}/run` | POST | CIController.java | 执行流水线 |
| `/api/devops/ci/executions/{id}` | GET | CIController.java | 获取执行状态 |

### 7.2 持续部署接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/devops/cd/deployments` | GET | CDController.java | 查询部署列表 |
| `/api/devops/cd/deployments` | POST | CDController.java | 创建部署 |
| `/api/devops/cd/deployments/{id}/deploy` | POST | CDController.java | 执行部署 |
| `/api/devops/cd/deployments/{id}/rollback` | POST | CDController.java | 回滚部署 |

### 7.3 测试管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/devops/test/cases` | GET | TestController.java | 查询测试用例 |
| `/api/devops/test/cases` | POST | TestController.java | 创建测试用例 |
| `/api/devops/test/run` | POST | TestController.java | 执行测试 |
| `/api/devops/test/reports` | GET | TestController.java | 获取测试报告 |

### 7.4 运维监控接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/devops/ops/metrics` | GET | OpsController.java | 获取监控指标 |
| `/api/devops/ops/logs` | GET | OpsController.java | 获取日志 |
| `/api/devops/ops/alerts` | GET | OpsController.java | 查询告警列表 |
| `/api/devops/ops/alerts` | POST | OpsController.java | 创建告警规则 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 流水线管理 | ci:read, ci:write | 查看和管理流水线 |
| 部署管理 | cd:read, cd:write | 查看和管理部署 |
| 运维监控 | ops:read, ops:write | 查看和管理运维 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| Jenkins | 2.4.x | CI/CD 工具 |
| Kubernetes | 1.28.x | 容器编排 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| ESP | REST API | 服务注册发现 |
| GitLab/GitHub | REST API | 代码仓库集成 |
| Kubernetes | Kubernetes API | 容器部署 |

---

**文档结束**