# Storage 存储服务平台设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 Storage 存储服务平台系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
Storage 系统作为企业级应用体系的统一存储服务平台，负责对象存储、文件存储、块存储、备份恢复等存储业务，与 DAM、DMS、各业务系统紧密协同，实现企业数据的统一存储和管理。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 对象存储 | 文件上传、下载、删除、版本管理 | 高 |
| 2 | 文件存储 | 目录管理、文件共享、权限控制 | 高 |
| 3 | 块存储 | 卷管理、快照、备份恢复 | 高 |
| 4 | 存储管理 | 存储配额、存储策略、存储监控 | 高 |
| 5 | 数据备份 | 备份计划、备份执行、恢复测试 | 高 |
| 6 | CDN 加速 | 静态资源加速、缓存策略、边缘分发 | 中 |
| 7 | 数据迁移 | 数据同步、跨云迁移、增量同步 | 中 |

### 2.2 非功能需求

| 类别 | 要求 |
| --- | --- |
| 性能 | 读写延迟 < 50ms，支持高并发 |
| 可用性 | 99.99% 高可用 |
| 安全性 | 符合等保 2.0 三级要求 |

---

## 3. 系统架构设计

### 3.1 架构风格
- **微服务架构**: 独立部署，高内聚低耦合
- **分布式存储**: 多副本冗余，数据分片

### 3.2 模块划分

| 模块 | 职责 | 说明 |
| --- | --- | --- |
| 对象存储模块 | 对象存储 | 上传、下载、版本管理 |
| 文件存储模块 | 文件存储 | 目录、共享、权限 |
| 块存储模块 | 块存储 | 卷管理、快照、备份 |
| 管理模块 | 存储管理 | 配额、策略、监控 |
| 备份模块 | 数据备份 | 备份、恢复、测试 |
| CDN 模块 | CDN 加速 | 加速、缓存、分发 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/storage/
  │   │   │   ├── controller/
  │   │   │   │   ├── ObjectStorageController.java # 对象存储
  │   │   │   │   ├── FileStorageController.java   # 文件存储
  │   │   │   │   ├── BlockStorageController.java  # 块存储
  │   │   │   │   ├── BackupController.java        # 备份管理
  │   │   │   │   └── CDNController.java           # CDN 管理
  │   │   │   ├── service/
  │   │   │   │   ├── ObjectStorageService.java
  │   │   │   │   ├── FileStorageService.java
  │   │   │   │   ├── BlockStorageService.java
  │   │   │   │   ├── BackupService.java
  │   │   │   │   └── CDNService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── StorageApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── storage/                         # 存储组件
  │   │   │   └── StorageCard.vue
  │   ├── views/
  │   │   ├── object/                          # 对象存储
  │   │   │   └── index.vue
  │   │   ├── file/                            # 文件存储
  │   │   │   └── index.vue
  │   │   ├── block/                           # 块存储
  │   │   │   └── index.vue
  │   │   ├── backup/                          # 备份管理
  │   │   │   └── index.vue
  │   │   └── cdn/                             # CDN 管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 ObjectStorageService (对象存储服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `uploadObject` | 上传对象 | `ObjectUploadRequest request` | `ObjectResponse` | 抛出`StorageException` |
| `downloadObject` | 下载对象 | `String bucketName, String objectKey` | `ObjectDownloadResponse` | 抛出`ObjectNotFoundException` |
| `deleteObject` | 删除对象 | `String bucketName, String objectKey` | `void` | 抛出`ObjectNotFoundException` |
| `listObjects` | 列出对象 | `ObjectListRequest request` | `List<ObjectResponse>` | - |

#### 5.1.2 FileStorageService (文件存储服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createDirectory` | 创建目录 | `DirectoryCreateRequest request` | `DirectoryResponse` | 抛出`BusinessException` |
| `uploadFile` | 上传文件 | `FileUploadRequest request` | `FileResponse` | 抛出`StorageException` |
| `shareFile` | 共享文件 | `FileShareRequest request` | `ShareResponse` | 抛出`FileNotFoundException` |
| `listFiles` | 列出文件 | `FileListRequest request` | `List<FileResponse>` | - |

#### 5.1.3 BackupService (备份服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createBackupPlan` | 创建备份计划 | `BackupPlanCreateRequest request` | `BackupPlanResponse` | 抛出`BusinessException` |
| `executeBackup` | 执行备份 | `Long planId` | `BackupExecutionResponse` | 抛出`BackupPlanNotFoundException` |
| `restoreBackup` | 恢复备份 | `Long backupId` | `BackupRestoreResponse` | 抛出`BackupNotFoundException` |
| `getBackupPlans` | 查询备份计划 | `BackupPlanSearchRequest request` | `Page<BackupPlanResponse>` | - |

### 5.2 DTO 结构定义

**ObjectUploadRequest（上传对象请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| bucketName | String | Bucket 名称 | 非空 |
| objectKey | String | 对象键 | 非空 |
| content | byte[] | 对象内容 | 非空 |
| contentType | String | 内容类型 | 非空 |
| metadata | Map<String, String> | 元数据 | 可选 |

**FileUploadRequest（上传文件请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| directoryId | Long | 目录 ID | 非空 |
| fileName | String | 文件名称 | 非空 |
| content | byte[] | 文件内容 | 非空 |
| contentType | String | 内容类型 | 非空 |

**BackupPlanCreateRequest（创建备份计划请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| planName | String | 计划名称 | 非空 |
| sourceType | String | 源类型 | 非空 |
| sourcePath | String | 源路径 | 非空 |
| backupType | String | 备份类型 | 非空 |
| scheduleType | String | 调度类型 | 非空 |
| cronExpression | String | Cron 表达式 | 非空 |
| retentionDays | Integer | 保留天数 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 Bucket 表 (storage_bucket)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | Bucket ID |
| bucket_name | VARCHAR(100) | UNIQUE, NOT NULL | Bucket 名称 |
| description | VARCHAR(500) | - | 描述 |
| storage_class | VARCHAR(50) | NOT NULL | 存储类型 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 对象表 (storage_object)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 对象 ID |
| bucket_id | BIGINT | FOREIGN KEY, NOT NULL | Bucket ID |
| object_key | VARCHAR(500) | NOT NULL | 对象键 |
| content_type | VARCHAR(100) | NOT NULL | 内容类型 |
| size | BIGINT | NOT NULL | 大小(字节) |
| version_id | VARCHAR(50) | - | 版本 ID |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 文件目录表 (storage_file_directory)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 目录 ID |
| parent_id | BIGINT | FOREIGN KEY | 父目录 ID |
| directory_name | VARCHAR(100) | NOT NULL | 目录名称 |
| path | VARCHAR(500) | NOT NULL | 完整路径 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 文件表 (storage_file)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 文件 ID |
| directory_id | BIGINT | FOREIGN KEY, NOT NULL | 目录 ID |
| file_name | VARCHAR(200) | NOT NULL | 文件名称 |
| file_extension | VARCHAR(20) | - | 文件扩展名 |
| size | BIGINT | NOT NULL | 大小(字节) |
| content_type | VARCHAR(100) | NOT NULL | 内容类型 |
| hash_value | VARCHAR(64) | - | 文件哈希值 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 备份计划表 (storage_backup_plan)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 计划 ID |
| plan_name | VARCHAR(100) | NOT NULL | 计划名称 |
| source_type | VARCHAR(50) | NOT NULL | 源类型 |
| source_path | VARCHAR(500) | NOT NULL | 源路径 |
| backup_type | VARCHAR(20) | NOT NULL | 备份类型 |
| schedule_type | VARCHAR(20) | NOT NULL | 调度类型 |
| cron_expression | VARCHAR(100) | NOT NULL | Cron 表达式 |
| retention_days | INT | NOT NULL | 保留天数 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 对象存储接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/storage/object/buckets` | GET | ObjectStorageController.java | 查询 Bucket 列表 |
| `/api/storage/object/buckets` | POST | ObjectStorageController.java | 创建 Bucket |
| `/api/storage/object/upload` | POST | ObjectStorageController.java | 上传对象 |
| `/api/storage/object/download` | GET | ObjectStorageController.java | 下载对象 |
| `/api/storage/object/delete` | DELETE | ObjectStorageController.java | 删除对象 |

### 7.2 文件存储接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/storage/file/directories` | GET | FileStorageController.java | 查询目录列表 |
| `/api/storage/file/directories` | POST | FileStorageController.java | 创建目录 |
| `/api/storage/file/upload` | POST | FileStorageController.java | 上传文件 |
| `/api/storage/file/share` | POST | FileStorageController.java | 共享文件 |
| `/api/storage/file/list` | GET | FileStorageController.java | 列出文件 |

### 7.3 块存储接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/storage/block/volumes` | GET | BlockStorageController.java | 查询卷列表 |
| `/api/storage/block/volumes` | POST | BlockStorageController.java | 创建卷 |
| `/api/storage/block/volumes/{id}/snapshot` | POST | BlockStorageController.java | 创建快照 |
| `/api/storage/block/volumes/{id}` | DELETE | BlockStorageController.java | 删除卷 |

### 7.4 备份管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/storage/backup/plans` | GET | BackupController.java | 查询备份计划 |
| `/api/storage/backup/plans` | POST | BackupController.java | 创建备份计划 |
| `/api/storage/backup/plans/{id}/execute` | POST | BackupController.java | 执行备份 |
| `/api/storage/backup/{id}/restore` | POST | BackupController.java | 恢复备份 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 对象存储 | object:read, object:write | 查看和管理对象 |
| 文件存储 | file:read, file:write | 查看和管理文件 |
| 备份管理 | backup:read, backup:write | 查看和管理备份 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.5.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| MySQL | 8.0+ | 数据库 |
| Redis | 7.0+ | 缓存 |
| MinIO | 8.5.x | 对象存储 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| DAM | REST API | 数字资产存储 |
| DMS | REST API | 文档存储 |

---

**文档结束**