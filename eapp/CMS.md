# CMS 内容管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 CMS（Content Management System）内容管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
CMS 系统作为企业级应用体系的内容管理平台，负责网站内容、营销内容、知识库内容的创建、编辑、发布和管理，与 MA 营销自动化系统、BI 数据分析系统紧密协同，提升企业内容运营效率。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 内容管理 | 文章、页面、媒体资源管理 | 高 |
| 2 | 内容编辑 | 富文本编辑、Markdown 编辑 | 高 |
| 3 | 内容发布 | 定时发布、草稿管理、版本控制 | 高 |
| 4 | 栏目管理 | 栏目分类、栏目权限 | 高 |
| 5 | 模板管理 | 页面模板、内容模板 | 中 |
| 6 | SEO 管理 | 关键词、Meta 信息、URL 优化 | 中 |
| 7 | 内容统计 | 访问量、阅读量、互动统计 | 中 |

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
| 内容模块 | 内容管理 | 文章、页面、媒体资源 |
| 编辑模块 | 内容编辑 | 富文本、Markdown 编辑器 |
| 发布模块 | 内容发布 | 定时发布、版本控制 |
| 栏目模块 | 栏目管理 | 栏目分类、权限 |
| 模板模块 | 模板管理 | 页面模板、内容模板 |
| 统计模块 | 内容统计 | 访问量、阅读量 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/cms/
  │   │   │   ├── controller/
  │   │   │   │   ├── ContentController.java    # 内容管理
  │   │   │   │   ├── CategoryController.java   # 栏目管理
  │   │   │   │   ├── TemplateController.java   # 模板管理
  │   │   │   │   └── MediaController.java      # 媒体资源管理
  │   │   │   ├── service/
  │   │   │   │   ├── ContentService.java
  │   │   │   │   ├── CategoryService.java
  │   │   │   │   ├── TemplateService.java
  │   │   │   │   └── MediaService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── CmsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── editor/                           # 编辑器组件
  │   │   │   ├── RichTextEditor.vue
  │   │   │   └── MarkdownEditor.vue
  │   ├── views/
  │   │   ├── content/                          # 内容管理
  │   │   │   ├── list.vue
  │   │   │   └── edit.vue
  │   │   ├── category/                         # 栏目管理
  │   │   │   └── index.vue
  │   │   ├── template/                         # 模板管理
  │   │   │   └── index.vue
  │   │   ├── media/                            # 媒体资源
  │   │   │   └── index.vue
  │   │   └── statistics/                       # 内容统计
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ContentService (内容服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createContent` | 创建内容 | `ContentCreateRequest request` | `ContentResponse` | 抛出`BusinessException` |
| `updateContent` | 更新内容 | `Long contentId, ContentUpdateRequest request` | `ContentResponse` | 抛出`ContentNotFoundException` |
| `publishContent` | 发布内容 | `Long contentId` | `ContentResponse` | 抛出`ContentNotFoundException` |
| `getContentVersions` | 获取内容版本 | `Long contentId` | `List<ContentVersionResponse>` | 抛出`ContentNotFoundException` |
| `getContents` | 查询内容列表 | `ContentSearchRequest request` | `Page<ContentResponse>` | - |

#### 5.1.2 MediaService (媒体服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `uploadMedia` | 上传媒体文件 | `MultipartFile file, MediaMeta meta` | `MediaResponse` | 抛出`UploadException` |
| `getMediaList` | 查询媒体列表 | `MediaSearchRequest request` | `Page<MediaResponse>` | - |
| `deleteMedia` | 删除媒体文件 | `Long mediaId` | `void` | 抛出`MediaNotFoundException` |

### 5.2 DTO 结构定义

**ContentCreateRequest（创建内容请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| title | String | 标题 | 非空 |
| content | String | 内容 | 非空 |
| categoryId | Long | 栏目 ID | 非空 |
| type | String | 内容类型(ARTICLE/PAGE) | 非空 |
| status | String | 状态(DRAFT/PUBLISHED) | 非空 |
| publishTime | LocalDateTime | 发布时间 | 可选 |
| seoKeywords | String | SEO 关键词 | 可选 |
| seoDescription | String | SEO 描述 | 可选 |

**MediaMeta（媒体元数据）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| name | String | 文件名称 | 非空 |
| category | String | 分类 | 可选 |
| description | String | 描述 | 可选 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 内容表 (cms_content)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 内容 ID |
| title | VARCHAR(200) | NOT NULL | 标题 |
| content | TEXT | NOT NULL | 内容 |
| category_id | BIGINT | FOREIGN KEY, NOT NULL | 栏目 ID |
| type | VARCHAR(20) | NOT NULL | 内容类型 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| publish_time | DATETIME | - | 发布时间 |
| seo_keywords | VARCHAR(200) | - | SEO 关键词 |
| seo_description | VARCHAR(500) | - | SEO 描述 |
| view_count | INT | DEFAULT 0 | 浏览次数 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.2 内容版本表 (cms_content_version)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 版本 ID |
| content_id | BIGINT | FOREIGN KEY, NOT NULL | 内容 ID |
| version | INT | NOT NULL | 版本号 |
| title | VARCHAR(200) | NOT NULL | 标题 |
| content | TEXT | NOT NULL | 内容 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 栏目表 (cms_category)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 栏目 ID |
| name | VARCHAR(100) | NOT NULL | 栏目名称 |
| code | VARCHAR(50) | UNIQUE, NOT NULL | 栏目编码 |
| parent_id | BIGINT | FOREIGN KEY | 父栏目 ID |
| level | INT | NOT NULL | 层级 |
| sort_order | INT | DEFAULT 0 | 排序 |
| status | VARCHAR(20) | DEFAULT 'ACTIVE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 媒体资源表 (cms_media)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 媒体 ID |
| name | VARCHAR(200) | NOT NULL | 文件名称 |
| file_path | VARCHAR(500) | NOT NULL | 文件路径 |
| file_size | BIGINT | NOT NULL | 文件大小 |
| file_type | VARCHAR(50) | NOT NULL | 文件类型 |
| category | VARCHAR(50) | - | 分类 |
| description | VARCHAR(200) | - | 描述 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 模板表 (cms_template)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 模板 ID |
| name | VARCHAR(100) | NOT NULL | 模板名称 |
| code | VARCHAR(50) | UNIQUE, NOT NULL | 模板编码 |
| type | VARCHAR(20) | NOT NULL | 模板类型 |
| content | TEXT | NOT NULL | 模板内容 |
| status | VARCHAR(20) | DEFAULT 'ACTIVE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 内容管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cms/content` | GET | ContentController.java | 查询内容列表 |
| `/api/cms/content/{id}` | GET | ContentController.java | 查询内容详情 |
| `/api/cms/content` | POST | ContentController.java | 创建内容 |
| `/api/cms/content/{id}` | PUT | ContentController.java | 更新内容 |
| `/api/cms/content/{id}/publish` | POST | ContentController.java | 发布内容 |

### 7.2 栏目管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cms/categories` | GET | CategoryController.java | 查询栏目列表 |
| `/api/cms/categories/tree` | GET | CategoryController.java | 查询栏目树 |
| `/api/cms/categories` | POST | CategoryController.java | 创建栏目 |

### 7.3 媒体资源接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cms/media` | GET | MediaController.java | 查询媒体列表 |
| `/api/cms/media` | POST | MediaController.java | 上传媒体文件 |
| `/api/cms/media/{id}` | DELETE | MediaController.java | 删除媒体文件 |

### 7.4 模板管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/cms/templates` | GET | TemplateController.java | 查询模板列表 |
| `/api/cms/templates` | POST | TemplateController.java | 创建模板 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 内容管理 | content:read, content:write | 查看和编辑内容 |
| 栏目管理 | category:read, category:write | 查看和编辑栏目 |
| 媒体管理 | media:read, media:upload | 查看和上传媒体 |

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

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| MA | REST API | 营销内容推送 |
| BI | REST API | 内容统计数据 |

---

**文档结束**