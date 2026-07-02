# AI 智能平台系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 AI（Artificial Intelligence）智能平台系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
AI 系统作为企业级应用体系的智能计算核心平台，负责模型训练、模型部署、推理服务、智能推荐等 AI 业务，与 CRM、BI、CDP 系统紧密协同，为企业各业务场景提供智能化能力支持。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 模型训练 | 数据准备、模型训练、模型评估 | 高 |
| 2 | 模型管理 | 模型版本、模型存储、模型发布 | 高 |
| 3 | 推理服务 | 在线推理、批量推理、实时预测 | 高 |
| 4 | 智能推荐 | 推荐算法、个性化推荐、推荐效果 | 高 |
| 5 | 自然语言处理 | 文本分析、情感分析、智能问答 | 高 |
| 6 | 图像识别 | 图像分类、目标检测、OCR 识别 | 中 |
| 7 | 智能分析 | 数据挖掘、趋势预测、异常检测 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 推理响应 < 100ms，支持高并发 |
| 可用性 | 99.9% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **分布式计算**: 支持分布式训练和推理

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 训练模块 | 模型训练 | 数据准备、训练、评估 |
| 管理模块 | 模型管理 | 版本、存储、发布 |
| 推理模块 | 推理服务 | 在线、批量、实时 |
| 推荐模块 | 智能推荐 | 算法、个性化、效果 |
| NLP模块 | 自然语言处理 | 文本、情感、问答 |
| CV模块 | 图像识别 | 分类、检测、OCR |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/ai/
  │   │   │   ├── controller/
  │   │   │   │   ├── ModelController.java       # 模型管理
  │   │   │   │   ├── TrainingController.java    # 模型训练
  │   │   │   │   ├── InferenceController.java   # 推理服务
  │   │   │   │   ├── RecommendationController.java # 智能推荐
  │   │   │   │   └── NLPController.java         # 自然语言处理
  │   │   │   ├── service/
  │   │   │   │   ├── ModelService.java
  │   │   │   │   ├── TrainingService.java
  │   │   │   │   ├── InferenceService.java
  │   │   │   │   ├── RecommendationService.java
  │   │   │   │   └── NLPService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── AiApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── model/                           # 模型组件
  │   │   │   └── ModelCard.vue
  │   ├── views/
  │   │   ├── models/                          # 模型管理
  │   │   │   └── index.vue
  │   │   ├── training/                        # 模型训练
  │   │   │   └── index.vue
  │   │   ├── inference/                       # 推理服务
  │   │   │   └── index.vue
  │   │   ├── recommendation/                  # 智能推荐
  │   │   │   └── index.vue
  │   │   └── nlp/                             # 自然语言处理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 TrainingService (训练服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `prepareData` | 准备训练数据 | `DataPreparationRequest request` | `DataPreparationResponse` | 抛出`DataPreparationException` |
| `trainModel` | 训练模型 | `ModelTrainingRequest request` | `ModelTrainingResponse` | 抛出`TrainingException` |
| `evaluateModel` | 评估模型 | `Long modelId` | `ModelEvaluationResponse` | 抛出`ModelNotFoundException` |
| `getTrainingJobs` | 查询训练任务 | `TrainingJobSearchRequest request` | `Page<TrainingJobResponse>` | - |

#### 5.1.2 ModelService (模型服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `registerModel` | 注册模型 | `ModelRegisterRequest request` | `ModelResponse` | 抛出`BusinessException` |
| `publishModel` | 发布模型 | `Long modelId` | `ModelResponse` | 抛出`ModelNotFoundException` |
| `getModelDetail` | 获取模型详情 | `Long modelId` | `ModelDetailResponse` | 抛出`ModelNotFoundException` |
| `getModels` | 查询模型列表 | `ModelSearchRequest request` | `Page<ModelResponse>` | - |

#### 5.1.3 InferenceService (推理服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `onlineInference` | 在线推理 | `OnlineInferenceRequest request` | `InferenceResponse` | 抛出`InferenceException` |
| `batchInference` | 批量推理 | `BatchInferenceRequest request` | `List<InferenceResponse>` | - |
| `realTimePredict` | 实时预测 | `PredictRequest request` | `PredictResponse` | 抛出`PredictException` |

### 5.2 DTO 结构定义

**ModelTrainingRequest（模型训练请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| modelName | String | 模型名称 | 非空 |
| modelType | String | 模型类型 | 非空 |
| algorithm | String | 算法名称 | 非空 |
| datasetId | Long | 数据集 ID | 非空 |
| hyperparameters | Map<String, Object> | 超参数 | 可选 |

**OnlineInferenceRequest（在线推理请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| modelId | Long | 模型 ID | 非空 |
| modelVersion | String | 模型版本 | 非空 |
| input | Map<String, Object> | 输入数据 | 非空 |

**RecommendationRequest（推荐请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| userId | Long | 用户 ID | 非空 |
| scene | String | 推荐场景 | 非空 |
| count | Integer | 推荐数量 | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 模型表 (ai_model)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 模型 ID |
| model_name | VARCHAR(100) | NOT NULL | 模型名称 |
| model_type | VARCHAR(50) | NOT NULL | 模型类型 |
| algorithm | VARCHAR(100) | NOT NULL | 算法名称 |
| version | VARCHAR(20) | NOT NULL | 版本号 |
| description | VARCHAR(500) | - | 描述 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| accuracy | DECIMAL(5,4) | - | 准确率 |
| file_path | VARCHAR(500) | - | 模型文件路径 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 训练任务表 (ai_training_job)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 任务 ID |
| job_name | VARCHAR(100) | NOT NULL | 任务名称 |
| model_id | BIGINT | FOREIGN KEY | 模型 ID |
| dataset_id | BIGINT | NOT NULL | 数据集 ID |
| algorithm | VARCHAR(100) | NOT NULL | 算法名称 |
| hyperparameters | TEXT | - | 超参数(JSON) |
| status | VARCHAR(20) | NOT NULL | 状态 |
| progress | INT | DEFAULT 0 | 进度 |
| start_time | DATETIME | - | 开始时间 |
| end_time | DATETIME | - | 结束时间 |
| error_message | TEXT | - | 错误信息 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 数据集表 (ai_dataset)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 数据集 ID |
| dataset_name | VARCHAR(100) | NOT NULL | 数据集名称 |
| dataset_type | VARCHAR(50) | NOT NULL | 数据集类型 |
| source | VARCHAR(100) | NOT NULL | 数据来源 |
| record_count | BIGINT | DEFAULT 0 | 记录数量 |
| file_path | VARCHAR(500) | NOT NULL | 文件路径 |
| description | VARCHAR(500) | - | 描述 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 推理记录表 (ai_inference_record)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 记录 ID |
| model_id | BIGINT | FOREIGN KEY, NOT NULL | 模型 ID |
| model_version | VARCHAR(20) | NOT NULL | 模型版本 |
| inference_type | VARCHAR(20) | NOT NULL | 推理类型 |
| input_data | TEXT | NOT NULL | 输入数据(JSON) |
| output_data | TEXT | - | 输出数据(JSON) |
| response_time | INT | NOT NULL | 响应时间(ms) |
| occurred_at | DATETIME | NOT NULL | 发生时间 |

#### 6.1.5 推荐记录表 (ai_recommendation_record)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 记录 ID |
| user_id | BIGINT | NOT NULL | 用户 ID |
| scene | VARCHAR(50) | NOT NULL | 推荐场景 |
| items | TEXT | NOT NULL | 推荐项(JSON) |
| clicked | BOOLEAN | DEFAULT FALSE | 是否点击 |
| converted | BOOLEAN | DEFAULT FALSE | 是否转化 |
| occurred_at | DATETIME | NOT NULL | 发生时间 |

---

## 7. API 接口设计

### 7.1 模型管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ai/models` | GET | ModelController.java | 查询模型列表 |
| `/api/ai/models/{id}` | GET | ModelController.java | 查询模型详情 |
| `/api/ai/models` | POST | ModelController.java | 注册模型 |
| `/api/ai/models/{id}/publish` | POST | ModelController.java | 发布模型 |

### 7.2 模型训练接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ai/training/jobs` | GET | TrainingController.java | 查询训练任务 |
| `/api/ai/training/jobs` | POST | TrainingController.java | 创建训练任务 |
| `/api/ai/training/jobs/{id}/stop` | POST | TrainingController.java | 停止训练任务 |
| `/api/ai/training/jobs/{id}/evaluate` | POST | TrainingController.java | 评估模型 |

### 7.3 推理服务接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ai/inference/online` | POST | InferenceController.java | 在线推理 |
| `/api/ai/inference/batch` | POST | InferenceController.java | 批量推理 |
| `/api/ai/inference/predict` | POST | InferenceController.java | 实时预测 |

### 7.4 智能推荐接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ai/recommendation` | POST | RecommendationController.java | 获取推荐 |
| `/api/ai/recommendation/feedback` | POST | RecommendationController.java | 推荐反馈 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 模型管理 | model:read, model:write | 查看和管理模型 |
| 训练管理 | training:read, training:write | 查看和管理训练 |
| 推理服务 | inference:read, inference:write | 查看和使用推理 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| TensorFlow | 2.13.x | 深度学习框架 |
| PyTorch | 2.0.x | 深度学习框架 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| CRM | REST API | 客户智能推荐 |
| BI | REST API | 智能分析 |
| CDP | REST API | 客户画像分析 |

---

**文档结束**