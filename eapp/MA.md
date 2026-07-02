# MA营销自动化系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述MA（Marketing Automation）营销自动化系统的设计方案，包括系统架构、功能模块、API接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
MA系统作为企业级应用体系的营销管理平台，负责营销活动管理、线索培育、客户分群、营销效果分析等营销业务，与CRM系统紧密协同，提升营销转化率和客户价值。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 活动管理 | 营销活动创建、执行、跟踪 | 高 |
| 2 | 线索管理 | 线索获取、评分、培育、转化 | 高 |
| 3 | 客户分群 | 客户标签、分群、画像 | 高 |
| 4 | 内容营销 | 营销内容管理、推送 | 高 |
| 5 | 渠道管理 | 多渠道营销（邮件、短信、社交媒体） | 高 |
| 6 | 营销自动化 | 自动化工作流、触发规则 | 高 |
| 7 | 效果分析 | 营销ROI、转化率分析 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 响应时间 < 200ms |
| 可用性 | 99.9%高可用 |
| 安全性 | 符合等保2.0三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **事件驱动**: 通过消息队列实现系统间协同

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 活动模块 | 营销活动管理 | 活动创建、执行、跟踪 |
| 线索模块 | 线索管理 | 线索获取、评分、培育 |
| 分群模块 | 客户分群 | 标签管理、客户分群 |
| 内容模块 | 内容营销 | 营销内容管理 |
| 渠道模块 | 渠道管理 | 多渠道营销 |
| 自动化模块 | 营销自动化 | 工作流、触发规则 |
| 分析模块 | 效果分析 | ROI、转化率分析 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/ma/
  │   │   │   ├── controller/
  │   │   │   │   ├── CampaignController.java    # 活动管理
  │   │   │   │   ├── LeadController.java        # 线索管理
  │   │   │   │   ├── SegmentController.java     # 客户分群
  │   │   │   │   ├── ContentController.java     # 内容营销
  │   │   │   │   ├── ChannelController.java     # 渠道管理
  │   │   │   │   └── AutomationController.java  # 营销自动化
  │   │   │   ├── service/
  │   │   │   │   ├── CampaignService.java
  │   │   │   │   ├── LeadService.java
  │   │   │   │   ├── SegmentService.java
  │   │   │   │   ├── ContentService.java
  │   │   │   │   ├── ChannelService.java
  │   │   │   │   └── AutomationService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   ├── automation/                    # 自动化引擎
  │   │   │   │   ├── WorkflowEngine.java
  │   │   │   │   └── TriggerRule.java
  │   │   │   └── MaApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── workflow/                          # 工作流组件
  │   │   │   ├── WorkflowDesigner.vue
  │   │   │   └── NodeConfig.vue
  │   ├── views/
  │   │   ├── campaign/                          # 活动管理
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── lead/                              # 线索管理
  │   │   │   ├── index.vue
  │   │   │   └── scoring.vue
  │   │   ├── segment/                           # 客户分群
  │   │   │   └── index.vue
  │   │   ├── content/                           # 内容营销
  │   │   │   └── index.vue
  │   │   ├── automation/                        # 营销自动化
  │   │   │   └── workflow.vue
  │   │   └── analysis/                          # 效果分析
  │   │       └── dashboard.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 CampaignService (活动服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createCampaign` | 创建活动 | `CampaignCreateRequest request` | `CampaignResponse` | 抛出`BusinessException` |
| `startCampaign` | 启动活动 | `Long campaignId` | `CampaignResponse` | 抛出`CampaignNotFoundException` |
| `stopCampaign` | 停止活动 | `Long campaignId` | `CampaignResponse` | 抛出`CampaignNotFoundException` |
| `getCampaigns` | 查询活动列表 | `CampaignSearchRequest request` | `Page<CampaignResponse>` | - |

#### 5.1.2 LeadService (线索服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createLead` | 创建线索 | `LeadCreateRequest request` | `LeadResponse` | 抛出`BusinessException` |
| `scoreLead` | 线索评分 | `Long leadId, Integer score` | `LeadResponse` | 抛出`LeadNotFoundException` |
| `convertLead` | 线索转化 | `Long leadId, ConvertRequest request` | `ConvertResult` | 抛出`LeadNotFoundException` |
| `getLeads` | 查询线索列表 | `LeadSearchRequest request` | `Page<LeadResponse>` | - |

#### 5.1.3 SegmentService (分群服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createSegment` | 创建分群 | `SegmentCreateRequest request` | `SegmentResponse` | 抛出`BusinessException` |
| `updateSegmentRules` | 更新分群规则 | `Long segmentId, List<Rule> rules` | `SegmentResponse` | 抛出`SegmentNotFoundException` |
| `getSegmentCustomers` | 获取分群客户 | `Long segmentId` | `List<CustomerResponse>` | 抛出`SegmentNotFoundException` |

#### 5.1.4 AutomationService (自动化服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createWorkflow` | 创建工作流 | `WorkflowCreateRequest request` | `WorkflowResponse` | 抛出`BusinessException` |
| `activateWorkflow` | 激活工作流 | `Long workflowId` | `WorkflowResponse` | 抛出`WorkflowNotFoundException` |
| `executeWorkflow` | 执行工作流 | `Long workflowId, String triggerEvent` | `WorkflowExecutionResult` | 抛出`WorkflowNotFoundException` |

### 5.2 DTO结构定义

**CampaignCreateRequest（创建活动请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| name | String | 活动名称 | 非空 |
| description | String | 活动描述 | 可选 |
| type | String | 活动类型 | 非空 |
| startDate | LocalDate | 开始日期 | 非空 |
| endDate | LocalDate | 结束日期 | 可选 |
| budget | BigDecimal | 预算金额 | 可选 |
| targetSegmentId | Long | 目标分群ID | 可选 |

**LeadCreateRequest（创建线索请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| source | String | 线索来源 | 非空 |
| name | String | 线索姓名 | 非空 |
| company | String | 公司名称 | 可选 |
| email | String | 邮箱 | 可选 |
| phone | String | 电话 | 可选 |
| score | Integer | 初始评分 | 可选，默认0 |

**WorkflowCreateRequest（创建工作流请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| name | String | 工作流名称 | 非空 |
| description | String | 工作流描述 | 可选 |
| triggerType | String | 触发类型 | 非空 |
| triggerCondition | String | 触发条件(JSON) | 非空 |
| actions | List<WorkflowAction> | 执行动作列表 | 非空 |

**WorkflowAction（工作流动作）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| actionType | String | 动作类型(SEND_EMAIL/SEND_SMS/CREATE_TASK) | 非空 |
| actionConfig | String | 动作配置(JSON) | 非空 |
| delayMinutes | Integer | 延迟时间 (分钟) | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 营销活动表 (ma_campaign)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 活动ID |
| name | VARCHAR(200) | NOT NULL | 活动名称 |
| description | VARCHAR(500) | - | 活动描述 |
| type | VARCHAR(50) | NOT NULL | 活动类型 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| start_date | DATE | NOT NULL | 开始日期 |
| end_date | DATE | - | 结束日期 |
| budget | DECIMAL(15,2) | - | 预算金额 |
| actual_cost | DECIMAL(15,2) | DEFAULT 0 | 实际成本 |
| created_by | BIGINT | NOT NULL | 创建人ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 线索表 (ma_lead)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 线索ID |
| source | VARCHAR(50) | NOT NULL | 线索来源 |
| name | VARCHAR(100) | NOT NULL | 线索姓名 |
| company | VARCHAR(200) | - | 公司名称 |
| email | VARCHAR(100) | - | 邮箱 |
| phone | VARCHAR(20) | - | 电话 |
| score | INT | DEFAULT 0 | 评分 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| owner_id | BIGINT | - | 负责人ID |
| converted_at | DATETIME | - | 转化时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 客户分群表 (ma_segment)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 分群ID |
| name | VARCHAR(200) | NOT NULL | 分群名称 |
| description | VARCHAR(500) | - | 分群描述 |
| rules | TEXT | NOT NULL | 分群规则(JSON) |
| customer_count | INT | DEFAULT 0 | 客户数量 |
| status | VARCHAR(20) | DEFAULT 'ACTIVE' | 状态 |
| created_by | BIGINT | NOT NULL | 创建人ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 营销内容表 (ma_content)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 内容ID |
| title | VARCHAR(200) | NOT NULL | 内容标题 |
| type | VARCHAR(50) | NOT NULL | 内容类型 |
| content | TEXT | NOT NULL | 内容正文 |
| template | TEXT | - | 模板内容 |
| status | VARCHAR(20) | DEFAULT 'DRAFT' | 状态 |
| created_by | BIGINT | NOT NULL | 创建人ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 自动化工作流表 (ma_workflow)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 工作流ID |
| name | VARCHAR(200) | NOT NULL | 工作流名称 |
| description | VARCHAR(500) | - | 工作流描述 |
| trigger_type | VARCHAR(50) | NOT NULL | 触发类型 |
| trigger_condition | TEXT | NOT NULL | 触发条件(JSON) |
| actions | TEXT | NOT NULL | 执行动作(JSON) |
| status | VARCHAR(20) | DEFAULT 'DRAFT' | 状态 |
| created_by | BIGINT | NOT NULL | 创建人ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.6 营销效果表 (ma_campaign_effect)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 效果ID |
| campaign_id | BIGINT | FOREIGN KEY, NOT NULL | 活动ID |
| date | DATE | NOT NULL | 日期 |
| impressions | INT | DEFAULT 0 | 曝光量 |
| clicks | INT | DEFAULT 0 | 点击量 |
| conversions | INT | DEFAULT 0 | 转化量 |
| cost | DECIMAL(15,2) | DEFAULT 0 | 成本 |
| revenue | DECIMAL(15,2) | DEFAULT 0 | 收入 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API接口设计

### 7.1 活动管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ma/campaigns` | GET | CampaignController.java | 查询活动列表 |
| `/api/ma/campaigns/{id}` | GET | CampaignController.java | 查询活动详情 |
| `/api/ma/campaigns` | POST | CampaignController.java | 创建活动 |
| `/api/ma/campaigns/{id}/start` | POST | CampaignController.java | 启动活动 |
| `/api/ma/campaigns/{id}/stop` | POST | CampaignController.java | 停止活动 |

### 7.2 线索管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ma/leads` | GET | LeadController.java | 查询线索列表 |
| `/api/ma/leads/{id}` | GET | LeadController.java | 查询线索详情 |
| `/api/ma/leads` | POST | LeadController.java | 创建线索 |
| `/api/ma/leads/{id}/score` | POST | LeadController.java | 线索评分 |
| `/api/ma/leads/{id}/convert` | POST | LeadController.java | 线索转化 |

### 7.3 客户分群接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ma/segments` | GET | SegmentController.java | 查询分群列表 |
| `/api/ma/segments/{id}` | GET | SegmentController.java | 查询分群详情 |
| `/api/ma/segments` | POST | SegmentController.java | 创建分群 |
| `/api/ma/segments/{id}/customers` | GET | SegmentController.java | 获取分群客户 |

### 7.4 营销自动化接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ma/workflows` | GET | AutomationController.java | 查询工作流列表 |
| `/api/ma/workflows/{id}` | GET | AutomationController.java | 查询工作流详情 |
| `/api/ma/workflows` | POST | AutomationController.java | 创建工作流 |
| `/api/ma/workflows/{id}/activate` | POST | AutomationController.java | 激活工作流 |

### 7.5 效果分析接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/ma/analysis/campaign/{id}` | GET | CampaignController.java | 获取活动效果 |
| `/api/ma/analysis/roi` | GET | CampaignController.java | 获取ROI分析 |
| `/api/ma/analysis/conversion` | GET | CampaignController.java | 获取转化率分析 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过SSO系统进行统一身份认证
- 使用JWT令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 活动管理 | campaign:read, campaign:write | 查看和编辑活动 |
| 线索管理 | lead:read, lead:write | 查看和编辑线索 |
| 客户分群 | segment:read, segment:write | 查看和编辑分群 |
| 营销自动化 | workflow:read, workflow:execute | 查看和执行工作流 |

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
| CRM | REST API + 消息队列 | 线索转化、客户同步 |

---

**文档结束**