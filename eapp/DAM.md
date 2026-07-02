# DAM 数字资产管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 DAM（Digital Asset Management）数字资产管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
DAM 系统作为企业级应用体系的数字资产管理平台，负责数字资产的存储、管理、检索、共享和分发，与 CMS、MA、DMS 系统紧密协同，实现企业数字资产的集中管理和高效利用。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 资产上传 | 批量上传、拖拽上传、版本管理 | 高 |
| 2 | 资产存储 | 多格式支持、分层存储、元数据管理 | 高 |
| 3 | 资产检索 | 全文检索、标签检索、智能推荐 | 高 |
| 4 | 资产共享 | 权限控制、分享链接、嵌入代码 | 高 |
| 5 | 资产分发 | 多渠道分发、格式转换、CDN加速 | 高 |
| 6 | 版权管理 | 版权信息、使用授权、到期提醒 | 中 |
| 7 | 资产分析 | 使用统计、热门排行、趋势分析 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 响应时间 < 200ms，支持海量文件存储 |
| 可用性 | 99.9% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **事件驱动**: 通过消息队列实现异步处理

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 上传模块 | 资产上传 | 批量上传、版本管理 |
| 存储模块 | 资产存储 | 多格式、元数据 |
| 检索模块 | 资产检索 | 全文检索、标签 |
| 共享模块 | 资产共享 | 权限、分享 |
| 分发模块 | 资产分发 | 多渠道、格式转换 |
| 版权模块 | 版权管理 | 版权、授权 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/dam/
  │   │   │   ├── controller/
  │   │   │   │   ├── AssetController.java       # 资产管理
  │   │   │   │   ├── SearchController.java      # 资产检索
  │   │   │   │   ├── ShareController.java       # 资产共享
  │   │   │   │   └── DistributionController.java # 资产分发
  │   │   │   ├── service/
  │   │   │   │   ├── AssetService.java
  │   │   │   │   ├── SearchService.java
  │   │   │   │   ├── ShareService.java
  │   │   │   │   └── DistributionService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── DamApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── gallery/                         # 画廊组件
  │   │   │   └── AssetGallery.vue
  │   ├── views/
  │   │   ├── asset/                           # 资产管理
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── search/                          # 资产检索
  │   │   │   └── index.vue
  │   │   ├── share/                           # 资产共享
  │   │   │   └── index.vue
  │   │   └── analysis/                        # 资产分析
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 AssetService (资产管理服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `uploadAsset` | 上传资产 | `AssetUploadRequest request` | `AssetResponse` | 抛出`UploadException` |
| `getAssetInfo` | 获取资产信息 | `Long assetId` | `AssetResponse` | 抛出`AssetNotFoundException` |
| `updateAsset` | 更新资产信息 | `Long assetId, AssetUpdateRequest request` | `AssetResponse` | 抛出`AssetNotFoundException` |
| `deleteAsset` | 删除资产 | `Long assetId` | `void` | 抛出`AssetNotFoundException` |

#### 5.1.2 SearchService (检索服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `searchAssets` | 搜索资产 | `AssetSearchRequest request` | `Page<AssetResponse>` | - |
| `searchByTags` | 按标签搜索 | `List<String> tags` | `Page<AssetResponse>` | - |
| `searchByContent` | 内容检索 | `String keyword` | `Page<AssetResponse>` | - |

#### 5.1.3 ShareService (共享服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createShareLink` | 创建分享链接 | `Long assetId, ShareRequest request` | `ShareLinkResponse` | 抛出`AssetNotFoundException` |
| `getSharedAssets` | 获取共享资产 | `Long userId` | `Page<ShareResponse>` | - |
| `revokeShare` | 撤销共享 | `Long shareId` | `void` | 抛出`ShareNotFoundException` |

### 5.2 DTO 结构定义

**AssetUploadRequest（上传资产请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| file | MultipartFile | 文件 | 非空 |
| name | String | 资产名称 | 非空 |
| description | String | 描述 | 可选 |
| tags | List<String> | 标签 | 可选 |
| categoryId | Long | 分类 ID | 可选 |
| copyright | String | 版权信息 | 可选 |

**AssetResponse（资产响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| id | Long | 资产 ID |
| name | String | 资产名称 |
| fileName | String | 文件名 |
| fileType | String | 文件类型 |
| fileSize | Long | 文件大小 |
| description | String | 描述 |
| tags | List<String> | 标签 |
| categoryId | Long | 分类 ID |
| thumbnailUrl | String | 缩略图地址 |
| downloadUrl | String | 下载地址 |
| status | String | 状态 |
| uploadedBy | Long | 上传人 ID |
| uploadedAt | LocalDateTime | 上传时间 |

**ShareRequest（共享请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| assetId | Long | 资产 ID | 非空 |
| shareType | String | 共享类型(PUBLIC/PRIVATE) | 非空 |
| expireAt | LocalDateTime | 过期时间 | 可选 |
| permissions | List<String> | 权限列表 | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 资产表 (dam_asset)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 资产 ID |
| name | VARCHAR(200) | NOT NULL | 资产名称 |
| file_name | VARCHAR(200) | NOT NULL | 文件名 |
| file_type | VARCHAR(50) | NOT NULL | 文件类型 |
| file_size | BIGINT | NOT NULL | 文件大小 |
| file_path | VARCHAR(500) | NOT NULL | 文件路径 |
| thumbnail_path | VARCHAR(500) | - | 缩略图路径 |
| description | VARCHAR(500) | - | 描述 |
| tags | VARCHAR(500) | - | 标签(JSON) |
| category_id | BIGINT | FOREIGN KEY | 分类 ID |
| copyright | VARCHAR(200) | - | 版权信息 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| uploaded_by | BIGINT | NOT NULL | 上传人 ID |
| uploaded_at | DATETIME | NOT NULL | 上传时间 |

#### 6.1.2 资产版本表 (dam_asset_version)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 版本 ID |
| asset_id | BIGINT | FOREIGN KEY, NOT NULL | 资产 ID |
| version | INT | NOT NULL | 版本号 |
| file_path | VARCHAR(500) | NOT NULL | 文件路径 |
| file_size | BIGINT | NOT NULL | 文件大小 |
| change_description | VARCHAR(200) | - | 变更说明 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 资产分类表 (dam_category)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 分类 ID |
| name | VARCHAR(100) | NOT NULL | 分类名称 |
| code | VARCHAR(50) | UNIQUE, NOT NULL | 分类编码 |
| parent_id | BIGINT | FOREIGN KEY | 父分类 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 资产共享表 (dam_share)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 共享 ID |
| asset_id | BIGINT | FOREIGN KEY, NOT NULL | 资产 ID |
| share_type | VARCHAR(20) | NOT NULL | 共享类型 |
| share_token | VARCHAR(100) | UNIQUE, NOT NULL | 共享令牌 |
| permissions | VARCHAR(200) | - | 权限(JSON) |
| expire_at | DATETIME | - | 过期时间 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 资产管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/dam/assets` | GET | AssetController.java | 查询资产列表 |
| `/api/dam/assets/{id}` | GET | AssetController.java | 查询资产详情 |
| `/api/dam/assets` | POST | AssetController.java | 上传资产 |
| `/api/dam/assets/{id}` | PUT | AssetController.java | 更新资产信息 |
| `/api/dam/assets/{id}` | DELETE | AssetController.java | 删除资产 |

### 7.2 资产检索接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/dam/search` | GET | SearchController.java | 搜索资产 |
| `/api/dam/search/tags` | GET | SearchController.java | 按标签搜索 |
| `/api/dam/search/content` | GET | SearchController.java | 内容检索 |

### 7.3 资产共享接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/dam/share` | POST | ShareController.java | 创建分享链接 |
| `/api/dam/share/{token}` | GET | ShareController.java | 通过链接访问 |
| `/api/dam/share/{id}` | DELETE | ShareController.java | 撤销共享 |

### 7.4 资产分发接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/dam/distribution/{id}/download` | GET | DistributionController.java | 下载资产 |
| `/api/dam/distribution/{id}/embed` | GET | DistributionController.java | 获取嵌入代码 |
| `/api/dam/distribution/{id}/convert` | POST | DistributionController.java | 格式转换 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 资产管理 | asset:read, asset:write | 查看和管理资产 |
| 资产共享 | share:read, share:write | 创建和管理共享 |
| 资产分发 | distribution:read, distribution:write | 下载和分发 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| MinIO | 8.5+ | 对象存储 |
| Elasticsearch | 8.x | 全文检索 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| CMS | REST API | 内容图片引用 |
| MA | REST API | 营销素材管理 |
| DMS | REST API | 文档附件管理 |

---

**文档结束**