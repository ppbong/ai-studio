# Kubernetes 容器编排平台设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 Kubernetes 容器编排平台系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
Kubernetes 系统作为企业级应用体系的容器编排核心平台，负责容器调度、资源管理、服务发现、自动扩缩容等容器化业务，与 DevOps、ESP、各业务系统紧密协同，实现应用的容器化部署和弹性伸缩。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 容器调度 | Pod 调度、节点管理、资源分配 | 高 |
| 2 | 服务管理 | 服务发现、负载均衡、服务网格 | 高 |
| 3 | 自动扩缩容 | HPA/VPA、自动伸缩、弹性调度 | 高 |
| 4 | 配置管理 | ConfigMap、Secret、配置注入 | 高 |
| 5 | 存储管理 | PV/PVC、存储类、持久化存储 | 高 |
| 6 | 网络管理 | 网络策略、Ingress、DNS | 中 |
| 7 | 监控告警 | 资源监控、健康检查、告警通知 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 支持万级 Pod 调度，调度延迟 < 1s |
| 可用性 | 99.99% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **分布式架构**: 多 Master + 多 Node 部署
- **声明式 API**: 声明式配置管理

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 调度模块 | 容器调度 | Pod 调度、节点管理 |
| 服务模块 | 服务管理 | 发现、负载均衡、网格 |
| 扩缩容模块 | 自动扩缩容 | HPA/VPA、弹性 |
| 配置模块 | 配置管理 | ConfigMap、Secret |
| 存储模块 | 存储管理 | PV/PVC、存储类 |
| 网络模块 | 网络管理 | 策略、Ingress、DNS |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/k8s/
  │   │   │   ├── controller/
  │   │   │   │   ├── ClusterController.java    # 集群管理
  │   │   │   │   ├── PodController.java        # Pod 管理
  │   │   │   │   ├── ServiceController.java    # 服务管理
  │   │   │   │   ├── DeploymentController.java # 部署管理
  │   │   │   │   └── ScaleController.java      # 扩缩容管理
  │   │   │   ├── service/
  │   │   │   │   ├── ClusterService.java
  │   │   │   │   ├── PodService.java
  │   │   │   │   ├── ServiceService.java
  │   │   │   │   ├── DeploymentService.java
  │   │   │   │   └── ScaleService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── KubernetesApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── cluster/                         # 集群组件
  │   │   │   └── ClusterCard.vue
  │   ├── views/
  │   │   ├── cluster/                         # 集群管理
  │   │   │   └── index.vue
  │   │   ├── pods/                            # Pod 管理
  │   │   │   └── index.vue
  │   │   ├── services/                        # 服务管理
  │   │   │   └── index.vue
  │   │   ├── deployments/                     # 部署管理
  │   │   │   └── index.vue
  │   │   └── scaling/                         # 扩缩容管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ClusterService (集群服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `getClusterInfo` | 获取集群信息 | `void` | `ClusterInfoResponse` | 抛出`ClusterException` |
| `getNodeList` | 获取节点列表 | `NodeSearchRequest request` | `List<NodeResponse>` | - |
| `getNodeDetail` | 获取节点详情 | `String nodeName` | `NodeDetailResponse` | 抛出`NodeNotFoundException` |
| `cordonNode` | 封锁节点 | `String nodeName` | `NodeResponse` | 抛出`NodeNotFoundException` |

#### 5.1.2 DeploymentService (部署服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createDeployment` | 创建部署 | `DeploymentCreateRequest request` | `DeploymentResponse` | 抛出`BusinessException` |
| `updateDeployment` | 更新部署 | `String name, DeploymentUpdateRequest request` | `DeploymentResponse` | 抛出`DeploymentNotFoundException` |
| `deleteDeployment` | 删除部署 | `String name, String namespace` | `void` | 抛出`DeploymentNotFoundException` |
| `getDeployments` | 查询部署列表 | `DeploymentSearchRequest request` | `List<DeploymentResponse>` | - |

#### 5.1.3 ScaleService (扩缩容服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `scaleDeployment` | 手动扩缩容 | `ScaleRequest request` | `ScaleResponse` | 抛出`DeploymentNotFoundException` |
| `createHPA` | 创建 HPA | `HPACreateRequest request` | `HPAResponse` | 抛出`BusinessException` |
| `getHPAStatus` | 获取 HPA 状态 | `String name, String namespace` | `HPAResponse` | 抛出`HPANotFoundException` |

### 5.2 DTO 结构定义

**DeploymentCreateRequest（创建部署请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| name | String | 部署名称 | 非空 |
| namespace | String | 命名空间 | 非空 |
| replicas | Integer | 副本数 | 非空 |
| image | String | 镜像地址 | 非空 |
| ports | List<ContainerPort> | 端口配置 | 可选 |

**ScaleRequest（扩缩容请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| name | String | 部署名称 | 非空 |
| namespace | String | 命名空间 | 非空 |
| replicas | Integer | 目标副本数 | 非空 |

**HPACreateRequest（创建 HPA 请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| name | String | HPA 名称 | 非空 |
| namespace | String | 命名空间 | 非空 |
| targetName | String | 目标资源名称 | 非空 |
| minReplicas | Integer | 最小副本数 | 非空 |
| maxReplicas | Integer | 最大副本数 | 非空 |
| metrics | List<MetricSpec> | 指标配置 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 集群表 (k8s_cluster)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 集群 ID |
| cluster_name | VARCHAR(100) | UNIQUE, NOT NULL | 集群名称 |
| kubeconfig | TEXT | NOT NULL | KubeConfig 配置 |
| version | VARCHAR(20) | NOT NULL | Kubernetes 版本 |
| status | VARCHAR(20) | NOT NULL | 集群状态 |
| node_count | INT | DEFAULT 0 | 节点数量 |
| pod_count | INT | DEFAULT 0 | Pod 数量 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 部署记录表 (k8s_deployment_record)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 记录 ID |
| cluster_id | BIGINT | FOREIGN KEY, NOT NULL | 集群 ID |
| deployment_name | VARCHAR(100) | NOT NULL | 部署名称 |
| namespace | VARCHAR(50) | NOT NULL | 命名空间 |
| replicas | INT | NOT NULL | 副本数 |
| image | VARCHAR(500) | NOT NULL | 镜像地址 |
| action_type | VARCHAR(20) | NOT NULL | 操作类型 |
| operator_id | BIGINT | NOT NULL | 操作人 ID |
| operated_at | DATETIME | NOT NULL | 操作时间 |

#### 6.1.3 节点表 (k8s_node)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 节点 ID |
| cluster_id | BIGINT | FOREIGN KEY, NOT NULL | 集群 ID |
| node_name | VARCHAR(100) | NOT NULL | 节点名称 |
| node_ip | VARCHAR(50) | NOT NULL | 节点 IP |
| status | VARCHAR(20) | NOT NULL | 节点状态 |
| role | VARCHAR(50) | NOT NULL | 节点角色 |
| cpu_total | DECIMAL(10,2) | NOT NULL | CPU 总量 |
| cpu_used | DECIMAL(10,2) | DEFAULT 0 | CPU 使用量 |
| memory_total | DECIMAL(15,2) | NOT NULL | 内存总量 |
| memory_used | DECIMAL(15,2) | DEFAULT 0 | 内存使用量 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.4 HPA 配置表 (k8s_hpa_config)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | HPA ID |
| cluster_id | BIGINT | FOREIGN KEY, NOT NULL | 集群 ID |
| hpa_name | VARCHAR(100) | NOT NULL | HPA 名称 |
| namespace | VARCHAR(50) | NOT NULL | 命名空间 |
| target_name | VARCHAR(100) | NOT NULL | 目标资源名称 |
| min_replicas | INT | NOT NULL | 最小副本数 |
| max_replicas | INT | NOT NULL | 最大副本数 |
| metrics | TEXT | NOT NULL | 指标配置(JSON) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 集群管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/k8s/cluster/info` | GET | ClusterController.java | 获取集群信息 |
| `/api/k8s/nodes` | GET | ClusterController.java | 获取节点列表 |
| `/api/k8s/nodes/{name}` | GET | ClusterController.java | 获取节点详情 |
| `/api/k8s/nodes/{name}/cordon` | POST | ClusterController.java | 封锁节点 |
| `/api/k8s/nodes/{name}/uncordon` | POST | ClusterController.java | 解封节点 |

### 7.2 部署管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/k8s/deployments` | GET | DeploymentController.java | 查询部署列表 |
| `/api/k8s/deployments/{namespace}/{name}` | GET | DeploymentController.java | 获取部署详情 |
| `/api/k8s/deployments` | POST | DeploymentController.java | 创建部署 |
| `/api/k8s/deployments/{namespace}/{name}` | PUT | DeploymentController.java | 更新部署 |
| `/api/k8s/deployments/{namespace}/{name}` | DELETE | DeploymentController.java | 删除部署 |

### 7.3 服务管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/k8s/services` | GET | ServiceController.java | 查询服务列表 |
| `/api/k8s/services/{namespace}/{name}` | GET | ServiceController.java | 获取服务详情 |
| `/api/k8s/services` | POST | ServiceController.java | 创建服务 |

### 7.4 扩缩容管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/k8s/scale` | POST | ScaleController.java | 手动扩缩容 |
| `/api/k8s/hpa` | GET | ScaleController.java | 查询 HPA 列表 |
| `/api/k8s/hpa` | POST | ScaleController.java | 创建 HPA |
| `/api/k8s/hpa/{namespace}/{name}` | GET | ScaleController.java | 获取 HPA 状态 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 KubeConfig 进行 Kubernetes API 访问

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 集群管理 | cluster:read, cluster:write | 查看和管理集群 |
| 部署管理 | deployment:read, deployment:write | 查看和管理部署 |
| 扩缩容管理 | scale:read, scale:write | 查看和管理扩缩容 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| Kubernetes | 1.28.x | 容器编排 |
| Fabric8 Kubernetes Client | 6.6.x | Kubernetes 客户端 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| DevOps | REST API | 持续部署集成 |
| ESP | REST API | 服务注册发现 |

---

**文档结束**