# CS客服系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述CS（Customer Service）客服系统的设计方案，包括系统架构、功能模块、API接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
CS系统作为企业级应用体系的客户服务平台，负责工单管理、在线客服、电话客服、客户反馈等客服业务，与CRM系统紧密协同，提升客户满意度。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 工单管理 | 工单创建、分配、处理、关闭 | 高 |
| 2 | 在线客服 | 实时聊天、消息记录、智能客服 | 高 |
| 3 | 电话客服 | 来电弹屏、通话记录、电话录音 | 高 |
| 4 | 客户反馈 | 反馈收集、分类处理、回复跟踪 | 高 |
| 5 | 知识库 | 常见问题、解决方案、知识分享 | 中 |
| 6 | 服务质量 | 满意度评价、服务质量统计 | 中 |
| 7 | 报表统计 | 工单报表、客服绩效报表 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 消息响应时间 < 100ms |
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
| 工单模块 | 工单管理 | 工单创建、分配、处理 |
| 在线模块 | 在线客服 | 实时聊天、智能客服 |
| 电话模块 | 电话客服 | 来电弹屏、通话记录 |
| 反馈模块 | 客户反馈 | 反馈收集、处理跟踪 |
| 知识库模块 | 知识库管理 | 常见问题、解决方案 |
| 报表模块 | 报表统计 | 工单报表、绩效统计 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/cs/
  │   │   │   ├── controller/
  │   │   │   │   ├── TicketController.java      # 工单管理
  │   │   │   │   ├── ChatController.java        # 在线客服
  │   │   │   │   ├── CallController.java        # 电话客服
  │   │   │   │   ├── FeedbackController.java    # 客户反馈
  │   │   │   │   └── KnowledgeController.java   # 知识库管理
  │   │   │   ├── service/
  │   │   │   │   ├── TicketService.java
  │   │   │   │   ├── ChatService.java
  │   │   │   │   ├── CallService.java
  │   │   │   │   ├── FeedbackService.java
  │   │   │   │   └── KnowledgeService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   ├── websocket/                     # WebSocket配置
  │   │   │   │   └── WebSocketConfig.java
  │   │   │   └── CsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── chat/                              # 聊天组件
  │   │   │   ├── ChatWindow.vue
  │   │   │   └── MessageList.vue
  │   ├── views/
  │   │   ├── ticket/                            # 工单管理
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── chat/                              # 在线客服
  │   │   │   └── index.vue
  │   │   ├── feedback/                          # 客户反馈
  │   │   │   └── index.vue
  │   │   └── knowledge/                         # 知识库管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 TicketService (工单服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createTicket` | 创建工单 | `TicketCreateRequest request` | `TicketResponse` | 抛出`BusinessException` |
| `assignTicket` | 分配工单 | `Long ticketId, Long agentId` | `TicketResponse` | 抛出`TicketNotFoundException` |
| `resolveTicket` | 解决工单 | `Long ticketId, String solution` | `TicketResponse` | 抛出`TicketNotFoundException` |
| `getTickets` | 查询工单列表 | `TicketSearchRequest request` | `Page<TicketResponse>` | - |

#### 5.1.2 ChatService (在线客服服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `startChat` | 开始聊天 | `Long customerId` | `ChatSessionResponse` | 抛出`BusinessException` |
| `sendMessage` | 发送消息 | `String sessionId, String content` | `ChatMessageResponse` | 抛出`SessionNotFoundException` |
| `endChat` | 结束聊天 | `String sessionId` | `ChatSessionResponse` | 抛出`SessionNotFoundException` |
| `getChatHistory` | 获取聊天记录 | `String sessionId` | `List<ChatMessageResponse>` | 抛出`SessionNotFoundException` |

#### 5.1.3 FeedbackService (反馈服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createFeedback` | 创建反馈 | `FeedbackCreateRequest request` | `FeedbackResponse` | 抛出`BusinessException` |
| `replyFeedback` | 回复反馈 | `Long feedbackId, String reply` | `FeedbackResponse` | 抛出`FeedbackNotFoundException` |
| `getFeedbacks` | 查询反馈列表 | `FeedbackSearchRequest request` | `Page<FeedbackResponse>` | - |

### 5.2 DTO结构定义

**TicketCreateRequest（创建工单请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| customerId | Long | 客户ID | 非空 |
| subject | String | 工单主题 | 非空 |
| content | String | 工单内容 | 非空 |
| priority | String | 优先级(LOW/MEDIUM/HIGH/URGENT) | 非空 |
| category | String | 工单分类 | 可选 |

**ChatMessageResponse（聊天消息响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| id | Long | 消息ID |
| sessionId | String | 会话ID |
| senderId | Long | 发送人ID |
| senderType | String | 发送人类型(CUSTOMER/AGENT) |
| content | String | 消息内容 |
| messageType | String | 消息类型(TEXT/IMAGE/FILE) |
| createdAt | LocalDateTime | 发送时间 |

**FeedbackCreateRequest（创建反馈请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| customerId | Long | 客户ID | 非空 |
| type | String | 反馈类型 | 非空 |
| content | String | 反馈内容 | 非空 |
| contactInfo | String | 联系方式 | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 工单表 (cs_ticket)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 工单ID |
| ticket_no | VARCHAR(50) | UNIQUE, NOT NULL | 工单号 |
| customer_id | BIGINT | NOT NULL | 客户ID |
| subject | VARCHAR(200) | NOT NULL | 工单主题 |
| content | TEXT | NOT NULL | 工单内容 |
| priority | VARCHAR(20) | NOT NULL | 优先级 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| category | VARCHAR(50) | - | 工单分类 |
| agent_id | BIGINT | - | 客服ID |
| created_at | DATETIME | NOT NULL | 创建时间 |
| resolved_at | DATETIME | - | 解决时间 |

#### 6.1.2 工单处理记录表 (cs_ticket_history)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 记录ID |
| ticket_id | BIGINT | FOREIGN KEY, NOT NULL | 工单ID |
| operator_id | BIGINT | NOT NULL | 操作人ID |
| action | VARCHAR(50) | NOT NULL | 操作类型 |
| content | TEXT | - | 操作内容 |
| created_at | DATETIME | NOT NULL | 操作时间 |

#### 6.1.3 聊天会话表 (cs_chat_session)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 会话ID |
| session_no | VARCHAR(50) | UNIQUE, NOT NULL | 会话编号 |
| customer_id | BIGINT | NOT NULL | 客户ID |
| agent_id | BIGINT | - | 客服ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| started_at | DATETIME | NOT NULL | 开始时间 |
| ended_at | DATETIME | - | 结束时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 聊天消息表 (cs_chat_message)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 消息ID |
| session_id | BIGINT | FOREIGN KEY, NOT NULL | 会话ID |
| sender_id | BIGINT | NOT NULL | 发送人ID |
| sender_type | VARCHAR(20) | NOT NULL | 发送人类型 |
| content | TEXT | NOT NULL | 消息内容 |
| message_type | VARCHAR(20) | NOT NULL | 消息类型 |
| created_at | DATETIME | NOT NULL | 发送时间 |

#### 6.1.5 客户反馈表 (cs_feedback)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 反馈ID |
| customer_id | BIGINT | NOT NULL | 客户ID |
| type | VARCHAR(50) | NOT NULL | 反馈类型 |
| content | TEXT | NOT NULL | 反馈内容 |
| contact_info | VARCHAR(100) | - | 联系方式 |
| status | VARCHAR(20) | DEFAULT 'PENDING' | 状态 |
| reply | TEXT | - | 回复内容 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| replied_at | DATETIME | - | 回复时间 |

#### 6.1.6 知识库表 (cs_knowledge)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 知识ID |
| title | VARCHAR(200) | NOT NULL | 知识标题 |
| category | VARCHAR(50) | NOT NULL | 知识分类 |
| content | TEXT | NOT NULL | 知识内容 |
| tags | VARCHAR(500) | - | 标签 |
| view_count | INT | DEFAULT 0 | 浏览次数 |
| status | VARCHAR(20) | DEFAULT 'PUBLISHED' | 状态 |
| created_by | BIGINT | NOT NULL | 创建人ID |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

---

## 7. API接口设计

### 7.1 工单管理接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cs/tickets` | GET | TicketController.java | 查询工单列表 |
| `/api/cs/tickets/{id}` | GET | TicketController.java | 查询工单详情 |
| `/api/cs/tickets` | POST | TicketController.java | 创建工单 |
| `/api/cs/tickets/{id}/assign` | POST | TicketController.java | 分配工单 |
| `/api/cs/tickets/{id}/resolve` | POST | TicketController.java | 解决工单 |

### 7.2 在线客服接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cs/chat/sessions` | POST | ChatController.java | 开始聊天 |
| `/api/cs/chat/sessions/{id}/messages` | POST | ChatController.java | 发送消息 |
| `/api/cs/chat/sessions/{id}` | DELETE | ChatController.java | 结束聊天 |
| `/api/cs/chat/sessions/{id}/history` | GET | ChatController.java | 获取聊天记录 |

### 7.3 客户反馈接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cs/feedbacks` | GET | FeedbackController.java | 查询反馈列表 |
| `/api/cs/feedbacks` | POST | FeedbackController.java | 创建反馈 |
| `/api/cs/feedbacks/{id}/reply` | POST | FeedbackController.java | 回复反馈 |

### 7.4 知识库接口

| API路径 | HTTP方法 | Controller文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cs/knowledge` | GET | KnowledgeController.java | 查询知识库 |
| `/api/cs/knowledge/{id}` | GET | KnowledgeController.java | 查询知识详情 |
| `/api/cs/knowledge` | POST | KnowledgeController.java | 创建知识 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过SSO系统进行统一身份认证
- 使用JWT令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 工单管理 | ticket:read, ticket:write | 查看和创建工单 |
| 在线客服 | chat:read, chat:send | 查看和发送消息 |
| 客户反馈 | feedback:read, feedback:reply | 查看和回复反馈 |
| 知识库 | knowledge:read, knowledge:write | 查看和编辑知识 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| WebSocket | - | 实时通信 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| CRM | REST API | 获取客户信息、同步工单 |

---

**文档结束**