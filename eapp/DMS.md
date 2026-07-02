# DMS 文档管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 DMS（Document Management System）文档管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
DMS 系统作为企业级应用体系的文档管理平台，负责文档创建、文档审批、版本管理、权限控制、文档检索等文档业务，与 OA、PLM、QMS 系统紧密协同，实现企业文档的集中管理和安全共享。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 文档管理 | 文档上传、下载、预览 | 高 |
| 2 | 文档审批 | 审批流程、审批历史 | 高 |
| 3 | 版本管理 | 版本控制、历史回溯 | 高 |
| 4 | 权限管理 | 文档权限、共享权限 | 高 |
| 5 | 文档检索 | 全文检索、分类检索 | 高 |
| 6 | 文档协作 | 在线编辑、批注评论 | 中 |
| 7 | 文档统计 | 访问统计、使用分析 | 中 |

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
| 文档模块 | 文档管理 | 上传、下载、预览 |
| 审批模块 | 文档审批 | 流程、历史 |
| 版本模块 | 版本管理 | 版本控制、回溯 |
| 权限模块 | 权限管理 | 权限、共享 |
| 检索模块 | 文档检索 | 全文、分类 |
| 协作模块 | 文档协作 | 编辑、批注 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/dms/
  │   │   │   ├── controller/
  │   │   │   │   ├── DocumentController.java    # 文档管理
  │   │   │   │   ├── ApprovalController.java   # 文档审批
  │   │   │   │   ├── VersionController.java    # 版本管理
  │   │   │   │   ├── PermissionController.java # 权限管理
  │   │   │   │   └── SearchController.java     # 文档检索
  │   │   │   ├── service/
  │   │   │   │   ├── DocumentService.java
  │   │   │   │   ├── ApprovalService.java
  │   │   │   │   ├── VersionService.java
  │   │   │   │   ├── PermissionService.java
  │   │   │   │   └── SearchService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── DmsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── viewer/                          # 文档预览组件
  │   │   │   └── DocumentViewer.vue
  │   ├── views/
  │   │   ├── document/                        # 文档管理
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── approval/                        # 文档审批
  │   │   │   └── index.vue
  │   │   ├── version/                        # 版本管理
  │   │   │   └── index.vue
  │   │   ├── search/                          # 文档检索
  │   │   │   └── index.vue
  │   │   └── recycle/                         # 回收站
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 DocumentService (文档服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `uploadDocument` | 上传文档 | `DocumentUploadRequest request` | `DocumentResponse` | 抛出`UploadException` |
| `downloadDocument` | 下载文档 | `Long documentId` | `byte[]` | 抛出`DocumentNotFoundException` |
| `previewDocument` | 预览文档 | `Long documentId` | `PreviewResponse` | 抛出`DocumentNotFoundException` |
| `deleteDocument` | 删除文档 | `Long documentId` | `void` | 抛出`DocumentNotFoundException` |
| `getDocuments` | 查询文档列表 | `DocumentSearchRequest request` | `Page<DocumentResponse>` | - |

#### 5.1.2 ApprovalService (审批服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `submitForApproval` | 提交审批 | `Long documentId, ApprovalRequest request` | `ApprovalResponse` | 抛出`DocumentNotFoundException` |
| `approveDocument` | 审批通过 | `Long approvalId` | `ApprovalResponse` | 抛出`ApprovalNotFoundException` |
| `rejectDocument` | 审批拒绝 | `Long approvalId, String reason` | `ApprovalResponse` | 抛出`ApprovalNotFoundException` |
| `getApprovalHistory` | 审批历史 | `Long documentId` | `List<ApprovalHistoryResponse>` | - |

#### 5.1.3 VersionService (版本服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createVersion` | 创建新版本 | `Long documentId` | `VersionResponse` | 抛出`DocumentNotFoundException` |
| `getVersions` | 获取版本列表 | `Long documentId` | `List<VersionResponse>` | - |
| `rollbackVersion` | 回滚版本 | `Long documentId, Long versionId` | `DocumentResponse` | 抛出`VersionNotFoundException` |

### 5.2 DTO 结构定义

**DocumentUploadRequest（上传文档请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| name | String | 文档名称 | 非空 |
| folderId | Long | 文件夹 ID | 可选 |
| categoryId | Long | 分类 ID | 可选 |
| file | MultipartFile | 文件 | 非空 |
| description | String | 描述 | 可选 |
| tags | List<String> | 标签 | 可选 |

**DocumentResponse（文档响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| id | Long | 文档 ID |
| name | String | 文档名称 |
| folderId | Long | 文件夹 ID |
| filePath | String | 文件路径 |
| fileSize | Long | 文件大小 |
| mimeType | String | 文件类型 |
| version | Integer | 当前版本 |
| status | String | 状态 |
| createdBy | Long | 创建人 ID |
| createdAt | LocalDateTime | 创建时间 |
| updatedAt | LocalDateTime | 更新时间 |

**ApprovalRequest（审批请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| approverId | Long | 审批人 ID | 非空 |
| deadline | LocalDateTime | 审批期限 | 可选 |
| comment | String | 审批意见 | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 文档表 (dms_document)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 文档 ID |
| name | VARCHAR(200) | NOT NULL | 文档名称 |
| folder_id | BIGINT | FOREIGN KEY | 文件夹 ID |
| category_id | BIGINT | FOREIGN KEY | 分类 ID |
| file_path | VARCHAR(500) | NOT NULL | 文件路径 |
| file_size | BIGINT | NOT NULL | 文件大小 |
| mime_type | VARCHAR(100) | NOT NULL | 文件类型 |
| version | INT | DEFAULT 1 | 当前版本 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| description | VARCHAR(500) | - | 描述 |
| tags | VARCHAR(500) | - | 标签(JSON) |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 文件夹表 (dms_folder)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 文件夹 ID |
| name | VARCHAR(100) | NOT NULL | 文件夹名称 |
| parent_id | BIGINT | FOREIGN KEY | 父文件夹 ID |
| owner_id | BIGINT | NOT NULL | 所有者 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 文档版本表 (dms_document_version)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 版本 ID |
| document_id | BIGINT | FOREIGN KEY, NOT NULL | 文档 ID |
| version | INT | NOT NULL | 版本号 |
| file_path | VARCHAR(500) | NOT NULL | 文件路径 |
| file_size | BIGINT | NOT NULL | 文件大小 |
| change_description | VARCHAR(200) | - | 变更说明 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 文档审批表 (dms_document_approval)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 审批 ID |
| document_id | BIGINT | FOREIGN KEY, NOT NULL | 文档 ID |
| workflow_id | BIGINT | NOT NULL | 流程 ID |
| approver_id | BIGINT | NOT NULL | 审批人 ID |
| status | VARCHAR(20) | NOT NULL | 审批状态 |
| comment | VARCHAR(500) | - | 审批意见 |
| approved_at | DATETIME | - | 审批时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 文档权限表 (dms_document_permission)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 权限 ID |
| document_id | BIGINT | FOREIGN KEY, NOT NULL | 文档 ID |
| principal_type | VARCHAR(20) | NOT NULL | 授权类型(USER/ROLE) |
| principal_id | BIGINT | NOT NULL | 授权对象 ID |
| permission | VARCHAR(50) | NOT NULL | 权限 |
| created_by | BIGINT | NOT NULL | 授权人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 文档管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/dms/documents` | GET | DocumentController.java | 查询文档列表 |
| `/api/dms/documents/{id}` | GET | DocumentController.java | 查询文档详情 |
| `/api/dms/documents` | POST | DocumentController.java | 上传文档 |
| `/api/dms/documents/{id}/download` | GET | DocumentController.java | 下载文档 |
| `/api/dms/documents/{id}/preview` | GET | DocumentController.java | 预览文档 |
| `/api/dms/documents/{id}` | DELETE | DocumentController.java | 删除文档 |

### 7.2 文档审批接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/dms/approval/submit` | POST | ApprovalController.java | 提交审批 |
| `/api/dms/approval/{id}/approve` | POST | ApprovalController.java | 审批通过 |
| `/api/dms/approval/{id}/reject` | POST | ApprovalController.java | 审批拒绝 |
| `/api/dms/approval/history/{documentId}` | GET | ApprovalController.java | 审批历史 |

### 7.3 版本管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/dms/versions/{documentId}` | GET | VersionController.java | 获取版本列表 |
| `/api/dms/versions/{documentId}/create` | POST | VersionController.java | 创建新版本 |
| `/api/dms/versions/{id}/rollback` | POST | VersionController.java | 回滚版本 |

### 7.4 文档检索接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/dms/search` | GET | SearchController.java | 全文检索 |
| `/api/dms/folders` | GET | SearchController.java | 查询文件夹 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 文档管理 | document:read, document:write | 查看和编辑文档 |
| 文档审批 | approval:read, approval:write | 审批文档 |
| 版本管理 | version:read, version:write | 版本控制 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| MinIO | 8.5+ | 文件存储 |
| Elasticsearch | 8.x | 全文检索 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| OA | REST API | 文档审批流程 |
| PLM | REST API | 技术文档同步 |
| QMS | REST API | 质量文档管理 |

---

**文档结束**