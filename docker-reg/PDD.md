# 产品设计文档 (PDD)

## 1. 系统架构设计

### 1.1 整体架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        Docker镜像管理系统                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    表现层 (Presentation)                  │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │  │
│  │  │  React UI   │  │ Docker API  │  │  Registry API   │  │  │
│  │  │  (Web界面)  │  │  (推送/拉取)│  │  (镜像管理)     │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    业务层 (Business)                      │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │  │
│  │  │仓库服务  │ │项目服务  │ │镜像服务  │ │用户服务  │    │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐    │  │
│  │  │角色服务  │ │权限服务  │ │日志服务  │ │系统服务  │    │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    数据层 (Data)                          │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐   │  │
│  │  │ SQLite3  │  │Memcached │  │  Local FileSystem    │   │  │
│  │  │ (元数据) │  │ (会话/缓存)│  │  (镜像存储)        │   │  │
│  │  └──────────┘  └──────────┘  └──────────────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 技术栈

| 层级 | 技术选型 | 说明 |
|------|----------|------|
| 前端 | React + TypeScript | 现代化Web界面 |
| 后端 | SpringBoot 3.x | 主应用框架 |
| 数据库 | SQLite3 | 轻量级关系型数据库 |
| 缓存 | Memcached | 会话管理和数据缓存 |
| 存储 | 本地文件系统 | 镜像层和配置文件存储 |
| 容器 | Docker | 容器化部署 |

### 1.3 模块划分

```
docker-reg/
├── docker-reg-ui/          # 前端React应用
├── docker-reg-api/         # 后端SpringBoot应用
│   ├── src/main/java/
│   │   └── com/dockerreg/
│   │       ├── controller/     # REST控制器
│   │       ├── service/        # 业务服务
│   │       ├── repository/     # 数据访问
│   │       ├── entity/         # 实体类
│   │       ├── dto/            # 数据传输对象
│   │       ├── config/         # 配置类
│   │       ├── security/       # 安全模块
│   │       ├── registry/       # Registry集成
│   │       └── exception/      # 异常处理
│   └── src/main/resources/
│       ├── application.yml     # 应用配置
│       └── schema.sql          # 数据库Schema
└── docker-reg-registry/    # Docker Registry服务
```

## 2. 数据库设计

### 2.1 ER图

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│   User      │       │   Role      │       │ Permission  │
├─────────────┤       ├─────────────┤       ├─────────────┤
│ id          │       │ id          │       │ id          │
│ username    │       │ name        │       │ name        │
│ password    │       │ description │       │ description │
│ email       │       │ is_system   │       │ resource    │
│ status      │       │ created_at  │       │ action      │
│ role_id     │◄──────│ updated_at  │       │ role_id     │
│ created_at  │       └─────────────┘       └─────────────┘
│ updated_at  │
│ last_login  │
│ must_change │
└─────────────┘

┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│ Repository  │       │   Project   │       │    Image    │
├─────────────┤       ├─────────────┤       ├─────────────┤
│ id          │◄──────│ id          │◄──────│ id          │
│ name        │       │ name        │       │ name        │
│ description │       │ repository_id│      │ project_id  │
│ is_default  │       │ visibility  │       │ description │
│ created_at  │       │ created_at  │       │ created_at  │
│ updated_at  │       │ updated_at  │       │ updated_at  │
└─────────────┘       └─────────────┘       └─────────────┘
                                                    │
                                                    ▼
                                            ┌─────────────┐
                                            │  ImageTag   │
                                            ├─────────────┤
                                            │ id          │
                                            │ image_id    │
                                            │ tag         │
                                            │ digest      │
                                            │ size        │
                                            │ pushed_by   │
                                            │ pushed_at   │
                                            └─────────────┘

┌─────────────┐
│ AuditLog    │
├─────────────┤
│ id          │
│ user_id     │
│ action      │
│ resource    │
│ resource_id │
│ ip_address  │
│ details     │
│ created_at  │
└─────────────┘
```

### 2.2 表结构设计

#### 2.2.1 用户表 (users)

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100),
    status TINYINT DEFAULT 1 COMMENT '0:禁用 1:启用',
    role_id INTEGER NOT NULL,
    must_change_pwd TINYINT DEFAULT 0 COMMENT '0:否 1:是',
    last_login DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (role_id) REFERENCES roles(id)
);
```

#### 2.2.2 角色表 (roles)

```sql
CREATE TABLE roles (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE,
    description VARCHAR(200),
    is_system TINYINT DEFAULT 0 COMMENT '0:否 1:是',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

#### 2.2.3 权限表 (permissions)

```sql
CREATE TABLE permissions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    role_id INTEGER NOT NULL,
    resource VARCHAR(50) NOT NULL COMMENT '资源类型: user,role,repo,project,image,system',
    action VARCHAR(50) NOT NULL COMMENT '操作: create,read,update,delete,push,pull',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (role_id) REFERENCES roles(id),
    UNIQUE(role_id, resource, action)
);
```

#### 2.2.4 仓库表 (repositories)

```sql
CREATE TABLE repositories (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE,
    description VARCHAR(500),
    is_default TINYINT DEFAULT 0 COMMENT '0:否 1:是',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

#### 2.2.5 项目表 (projects)

```sql
CREATE TABLE projects (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(100) NOT NULL,
    repository_id INTEGER NOT NULL,
    visibility VARCHAR(20) DEFAULT 'private' COMMENT 'public/private',
    description VARCHAR(500),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (repository_id) REFERENCES repositories(id),
    UNIQUE(repository_id, name)
);
```

#### 2.2.6 镜像表 (images)

```sql
CREATE TABLE images (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(200) NOT NULL,
    project_id INTEGER NOT NULL,
    description TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (project_id) REFERENCES projects(id),
    UNIQUE(project_id, name)
);
```

#### 2.2.7 镜像标签表 (image_tags)

```sql
CREATE TABLE image_tags (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    image_id INTEGER NOT NULL,
    tag VARCHAR(100) NOT NULL,
    digest VARCHAR(255) NOT NULL,
    size BIGINT DEFAULT 0,
    manifest TEXT,
    pushed_by INTEGER,
    pushed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (image_id) REFERENCES images(id),
    FOREIGN KEY (pushed_by) REFERENCES users(id),
    UNIQUE(image_id, tag)
);
```

#### 2.2.8 审计日志表 (audit_logs)

```sql
CREATE TABLE audit_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER,
    username VARCHAR(50),
    action VARCHAR(50) NOT NULL,
    resource VARCHAR(50) NOT NULL,
    resource_id VARCHAR(50),
    ip_address VARCHAR(50),
    user_agent VARCHAR(500),
    details TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

#### 2.2.9 系统配置表 (system_config)

```sql
CREATE TABLE system_config (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    config_key VARCHAR(100) NOT NULL UNIQUE,
    config_value TEXT,
    description VARCHAR(200),
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 2.3 初始化数据

```sql
-- 初始化默认角色
INSERT INTO roles (name, description, is_system) VALUES 
('admin', '系统管理员', 1),
('developer', '开发者', 1),
('guest', '访客', 1);

-- 初始化管理员用户 (密码: admin123，需要首次登录修改)
INSERT INTO users (username, password, email, role_id, must_change_pwd, status) VALUES 
('admin', '$2a$10$...', 'admin@localhost', 1, 1, 1);

-- 初始化默认仓库
INSERT INTO repositories (name, description, is_default) VALUES 
('library', '默认仓库', 1);

-- 初始化权限
-- 管理员权限
INSERT INTO permissions (role_id, resource, action) VALUES 
(1, 'user', 'create'), (1, 'user', 'read'), (1, 'user', 'update'), (1, 'user', 'delete'),
(1, 'role', 'create'), (1, 'role', 'read'), (1, 'role', 'update'), (1, 'role', 'delete'),
(1, 'repository', 'create'), (1, 'repository', 'read'), (1, 'repository', 'update'), (1, 'repository', 'delete'),
(1, 'project', 'create'), (1, 'project', 'read'), (1, 'project', 'update'), (1, 'project', 'delete'),
(1, 'image', 'create'), (1, 'image', 'read'), (1, 'image', 'update'), (1, 'image', 'delete'),
(1, 'image', 'push'), (1, 'image', 'pull'),
(1, 'system', 'config'), (1, 'system', 'log');

-- 开发者权限
INSERT INTO permissions (role_id, resource, action) VALUES 
(2, 'image', 'read'), (2, 'image', 'push'), (2, 'image', 'pull'),
(2, 'image', 'update'), (2, 'image', 'delete'),
(2, 'project', 'read');

-- 访客权限
INSERT INTO permissions (role_id, resource, action) VALUES 
(3, 'image', 'read'), (3, 'image', 'pull'),
(3, 'project', 'read');
```

## 3. 接口设计

### 3.1 接口规范

**基础路径：** `/api/v1`

**请求格式：**
- Content-Type: application/json
- Authorization: Bearer {token}

**响应格式：**
```json
{
    "code": 200,
    "message": "success",
    "data": {}
}
```

**错误码：**
| 错误码 | 说明 |
|--------|------|
| 200 | 成功 |
| 400 | 请求参数错误 |
| 401 | 未认证 |
| 403 | 无权限 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |

### 3.2 认证接口

#### 3.2.1 用户登录

```
POST /api/v1/auth/login

Request:
{
    "username": "admin",
    "password": "admin123"
}

Response:
{
    "code": 200,
    "message": "success",
    "data": {
        "token": "eyJhbGciOiJIUzI1NiIs...",
        "user": {
            "id": 1,
            "username": "admin",
            "email": "admin@localhost",
            "role": "admin",
            "mustChangePwd": true
        }
    }
}
```

#### 3.2.2 修改密码

```
PUT /api/v1/auth/password

Request:
{
    "oldPassword": "admin123",
    "newPassword": "newPassword123"
}

Response:
{
    "code": 200,
    "message": "密码修改成功",
    "data": null
}
```

#### 3.2.3 获取当前用户

```
GET /api/v1/auth/current

Response:
{
    "code": 200,
    "message": "success",
    "data": {
        "id": 1,
        "username": "admin",
        "email": "admin@localhost",
        "role": {
            "id": 1,
            "name": "admin"
        },
        "mustChangePwd": false
    }
}
```

### 3.3 用户管理接口

#### 3.3.1 获取用户列表

```
GET /api/v1/users?page=1&size=10&keyword=admin

Response:
{
    "code": 200,
    "data": {
        "total": 10,
        "list": [
            {
                "id": 1,
                "username": "admin",
                "email": "admin@localhost",
                "status": 1,
                "role": {"id": 1, "name": "admin"},
                "lastLogin": "2024-01-01 10:00:00",
                "createdAt": "2024-01-01 00:00:00"
            }
        ]
    }
}
```

#### 3.3.2 创建用户

```
POST /api/v1/users

Request:
{
    "username": "user1",
    "password": "password123",
    "email": "user1@example.com",
    "roleId": 2
}

Response:
{
    "code": 200,
    "message": "创建成功",
    "data": {"id": 2}
}
```

#### 3.3.3 更新用户

```
PUT /api/v1/users/{id}

Request:
{
    "email": "newemail@example.com",
    "status": 1,
    "roleId": 2
}
```

#### 3.3.4 删除用户

```
DELETE /api/v1/users/{id}
```

### 3.4 仓库管理接口

#### 3.4.1 获取仓库列表

```
GET /api/v1/repositories

Response:
{
    "code": 200,
    "data": [
        {
            "id": 1,
            "name": "library",
            "description": "默认仓库",
            "isDefault": true,
            "projectCount": 5,
            "createdAt": "2024-01-01 00:00:00"
        }
    ]
}
```

#### 3.4.2 创建仓库

```
POST /api/v1/repositories

Request:
{
    "name": "myrepo",
    "description": "我的仓库"
}
```

#### 3.4.3 更新仓库

```
PUT /api/v1/repositories/{id}

Request:
{
    "description": "更新后的描述"
}
```

#### 3.4.4 删除仓库

```
DELETE /api/v1/repositories/{id}
```

### 3.5 项目管理接口

#### 3.5.1 获取项目列表

```
GET /api/v1/repositories/{repoId}/projects?visibility=public

Response:
{
    "code": 200,
    "data": [
        {
            "id": 1,
            "name": "mysql",
            "repository": {"id": 1, "name": "library"},
            "visibility": "public",
            "imageCount": 2,
            "createdAt": "2024-01-01 00:00:00"
        }
    ]
}
```

#### 3.5.2 创建项目

```
POST /api/v1/repositories/{repoId}/projects

Request:
{
    "name": "mysql",
    "visibility": "public",
    "description": "MySQL数据库镜像"
}
```

#### 3.5.3 更新项目

```
PUT /api/v1/projects/{id}

Request:
{
    "visibility": "private",
    "description": "更新后的描述"
}
```

#### 3.5.4 删除项目

```
DELETE /api/v1/projects/{id}
```

### 3.6 镜像管理接口

#### 3.6.1 获取镜像列表

```
GET /api/v1/projects/{projectId}/images

Response:
{
    "code": 200,
    "data": [
        {
            "id": 1,
            "name": "mysql",
            "project": {"id": 1, "name": "mysql"},
            "description": "MySQL数据库",
            "tags": [
                {
                    "tag": "5.7",
                    "digest": "sha256:...",
                    "size": 450000000,
                    "pushedBy": "admin",
                    "pushedAt": "2024-01-01 10:00:00"
                },
                {
                    "tag": "8.0",
                    "digest": "sha256:...",
                    "size": 550000000,
                    "pushedBy": "admin",
                    "pushedAt": "2024-01-02 10:00:00"
                }
            ],
            "createdAt": "2024-01-01 00:00:00"
        }
    ]
}
```

#### 3.6.2 更新镜像描述

```
PUT /api/v1/images/{id}

Request:
{
    "description": "MySQL 8.0 数据库镜像"
}
```

#### 3.6.3 删除镜像

```
DELETE /api/v1/images/{id}
```

#### 3.6.4 修改镜像标签

```
PUT /api/v1/images/{id}/tags/{tagId}

Request:
{
    "newTag": "latest"
}
```

#### 3.6.5 删除镜像标签

```
DELETE /api/v1/images/{id}/tags/{tagId}
```

### 3.7 Docker Registry API 集成

#### 3.7.1 镜像推送认证

```
POST /api/v1/registry/auth

Request:
{
    "account": "admin",
    "scope": "repository:library/mysql:push,pull"
}

Response:
{
    "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

#### 3.7.2 镜像清单获取

```
GET /v2/{repository}/{image}/manifests/{tag}

Headers:
Authorization: Bearer {token}
Accept: application/vnd.docker.distribution.manifest.v2+json
```

#### 3.7.3 镜像层获取

```
GET /v2/{repository}/{image}/blobs/{digest}

Headers:
Authorization: Bearer {token}
```

### 3.8 系统管理接口

#### 3.8.1 获取系统配置

```
GET /api/v1/system/config

Response:
{
    "code": 200,
    "data": {
        "registryDomain": "registry.example.com",
        "storagePath": "/data/registry",
        "maxUploadSize": "10GB"
    }
}
```

#### 3.8.2 更新系统配置

```
PUT /api/v1/system/config

Request:
{
    "registryDomain": "registry.example.com",
    "maxUploadSize": "20GB"
}
```

#### 3.8.3 获取审计日志

```
GET /api/v1/system/audit-logs?page=1&size=20&action=push&resource=image

Response:
{
    "code": 200,
    "data": {
        "total": 100,
        "list": [
            {
                "id": 1,
                "username": "admin",
                "action": "push",
                "resource": "image",
                "resourceId": "library/mysql:5.7",
                "ipAddress": "192.168.1.100",
                "details": "推送镜像 mysql:5.7",
                "createdAt": "2024-01-01 10:00:00"
            }
        ]
    }
}
```

## 4. 安全设计

### 4.1 认证机制

```
┌─────────┐     ┌─────────┐     ┌─────────┐
│  Client │────►│   API   │────►│  Auth   │
│         │     │ Gateway │     │ Service │
└─────────┘     └─────────┘     └─────────┘
                      │
                      ▼
                ┌─────────┐
                │ JWT验证 │
                └─────────┘
```

**JWT Token结构：**
```json
{
    "sub": "1",
    "username": "admin",
    "role": "admin",
    "iat": 1704067200,
    "exp": 1704153600
}
```

**Token有效期：**
- Access Token: 2小时
- Refresh Token: 7天

### 4.2 权限控制

**RBAC模型：**
```
用户 ──► 角色 ──► 权限
```

**权限校验流程：**
1. 解析JWT Token获取用户角色
2. 查询角色对应的权限列表
3. 校验请求资源+操作是否在权限列表中
4. 返回校验结果

### 4.3 密码安全

- 使用BCrypt加密存储
- 密码强度要求：至少8位，包含大小写字母和数字
- 首次登录强制修改密码
- 密码修改需要验证旧密码

### 4.4 传输安全

- 支持HTTPS/TLS加密
- Docker Registry支持TLS客户端认证
- API接口强制使用HTTPS

### 4.5 审计日志

记录以下操作：
- 用户登录/登出
- 用户/角色/权限变更
- 仓库/项目/镜像增删改
- 镜像推送/拉取
- 系统配置变更

## 5. 存储设计

### 5.1 镜像存储结构

```
/data/registry/
├── docker/
│   └── registry/
│       └── v2/
│           ├── blobs/
│           │   └── sha256/
│           │       ├── ab/
│           │       │   └── abcdef123456...
│           │       └── cd/
│           │           └── cdef12345678...
│           └── repositories/
│               └── library/
│                   └── mysql/
│                       ├── _layers/
│                       │   └── sha256/
│                       │       └── abcdef123456.../link
│                       ├── _manifests/
│                       │   ├── revisions/
│                       │   │   └── sha256/
│                       │   │       └── 1234567890ab.../link
│                       │   └── tags/
│                       │       ├── 5.7/
│                       │       │   └── current/link
│                       │       └── 8.0/
│                       │           └── current/link
│                       └── _uploads/
```

### 5.2 缓存策略

**Memcached缓存项：**
| Key | Value | TTL | 说明 |
|-----|-------|-----|------|
| session:{token} | UserSession | 2h | 用户会话 |
| user:{id} | UserInfo | 1h | 用户信息 |
| role:{id} | RoleInfo | 1h | 角色信息 |
| repo:list | List<Repository> | 10m | 仓库列表 |
| project:{repoId} | List<Project> | 10m | 项目列表 |

### 5.3 备份策略

**数据库备份：**
- 每日凌晨自动备份SQLite数据库文件
- 保留最近7天的备份
- 备份文件压缩存储

**镜像备份：**
- 支持增量备份
- 支持导出到外部存储

## 6. 部署设计

### 6.1 单机部署架构

```
┌─────────────────────────────────────────────────────────┐
│                    Docker Host                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              docker-reg 容器                     │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐ │   │
│  │  │  Nginx  │  │  Java   │  │   Registry      │ │   │
│  │  │ (80/443)│  │  (8080) │  │    (5000)       │ │   │
│  │  └─────────┘  └─────────┘  └─────────────────┘ │   │
│  │  ┌─────────┐  ┌─────────┐                      │   │
│  │  │Memcached│  │ SQLite  │                      │   │
│  │  │ (11211) │  │  (文件) │                      │   │
│  │  └─────────┘  └─────────┘                      │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │              数据卷挂载                          │   │
│  │  /data/registry    - 镜像存储                    │   │
│  │  /data/db          - 数据库文件                  │   │
│  │  /data/config      - 配置文件                    │   │
│  │  /data/logs        - 日志文件                    │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 6.2 Docker Compose配置

```yaml
version: '3.8'

services:
  docker-reg:
    image: docker-reg:latest
    container_name: docker-reg
    ports:
      - "80:80"
      - "443:443"
      - "5000:5000"
    volumes:
      - /data/registry:/data/registry
      - /data/db:/data/db
      - /data/config:/data/config
      - /data/logs:/data/logs
    environment:
      - REGISTRY_DOMAIN=registry.example.com
      - ADMIN_PASSWORD=admin123
      - JAVA_OPTS=-Xmx2g -Xms1g
    restart: unless-stopped

  memcached:
    image: memcached:alpine
    container_name: docker-reg-memcached
    ports:
      - "11211:11211"
    restart: unless-stopped
```

### 6.3 系统要求

| 资源 | 最低配置 | 推荐配置 |
|------|----------|----------|
| CPU | 2核 | 4核 |
| 内存 | 4GB | 8GB |
| 磁盘 | 50GB | 200GB+ |
| 网络 | 100Mbps | 1Gbps |

## 7. 前端设计

### 7.1 页面结构

```
src/
├── pages/
│   ├── login/              # 登录页
│   ├── dashboard/          # 仪表盘
│   ├── repository/         # 仓库管理
│   │   ├── list/
│   │   └── detail/
│   ├── project/            # 项目管理
│   │   ├── list/
│   │   └── detail/
│   ├── image/              # 镜像管理
│   │   ├── list/
│   │   └── detail/
│   ├── user/               # 用户管理
│   ├── role/               # 角色管理
│   ├── system/             # 系统设置
│   │   ├── config/
│   │   └── audit/
│   └── profile/            # 个人中心
├── components/
│   ├── layout/             # 布局组件
│   ├── common/             # 通用组件
│   └── business/           # 业务组件
└── services/
    ├── api.ts              # API封装
    └── auth.ts             # 认证服务
```

### 7.2 路由设计

```typescript
const routes = [
    { path: '/login', component: Login },
    { 
        path: '/',
        component: Layout,
        children: [
            { path: '', component: Dashboard },
            { path: 'repositories', component: RepositoryList },
            { path: 'repositories/:id', component: RepositoryDetail },
            { path: 'projects/:id', component: ProjectDetail },
            { path: 'images/:id', component: ImageDetail },
            { path: 'users', component: UserList },
            { path: 'roles', component: RoleList },
            { path: 'system/config', component: SystemConfig },
            { path: 'system/audit', component: AuditLog },
            { path: 'profile', component: Profile }
        ]
    }
];
```

### 7.3 状态管理

使用React Context + Hooks进行状态管理：

```typescript
// 全局状态
interface AppState {
    user: UserInfo | null;
    permissions: string[];
    theme: 'light' | 'dark';
}

// Context
const AppContext = createContext<AppState>(initialState);
```

## 8. 异常处理

### 8.1 异常分类

| 异常类型 | 处理方式 |
|----------|----------|
| 业务异常 | 返回具体错误码和消息 |
| 参数异常 | 返回400错误 |
| 认证异常 | 返回401，跳转登录页 |
| 权限异常 | 返回403，提示无权限 |
| 系统异常 | 返回500，记录日志 |

### 8.2 错误响应格式

```json
{
    "code": 40001,
    "message": "用户名或密码错误",
    "timestamp": "2024-01-01T10:00:00Z",
    "path": "/api/v1/auth/login"
}
```

### 8.3 全局异常处理

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException e) {
        // 业务异常处理
    }
    
    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ErrorResponse> handleAuthException(AuthenticationException e) {
        // 认证异常处理
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception e) {
        // 系统异常处理
    }
}
```

## 9. 性能优化

### 9.1 数据库优化

- 为常用查询字段建立索引
- 使用连接池管理数据库连接
- 分页查询避免全表扫描
- 定期清理审计日志

### 9.2 缓存优化

- 热点数据缓存到Memcached
- 设置合理的缓存过期时间
- 缓存穿透保护（空值缓存）
- 缓存雪崩防护（随机过期时间）

### 9.3 接口优化

- 接口响应压缩
- 静态资源CDN加速
- 图片懒加载
- 列表数据分页加载

### 9.4 镜像推送优化

- 支持分块上传
- 断点续传
- 并行层下载

## 10. 监控与日志

### 10.1 应用监控

- JVM内存使用情况
- 接口响应时间
- 错误率统计
- 并发用户数

### 10.2 业务监控

- 镜像推送/拉取次数
- 存储空间使用情况
- 用户活跃度
- 仓库/项目数量统计

### 10.3 日志规范

**日志级别：**
- ERROR: 系统错误
- WARN: 警告信息
- INFO: 重要业务日志
- DEBUG: 调试信息

**日志格式：**
```
[时间] [级别] [线程] [类名] - 消息内容
```

**日志文件：**
- application.log: 应用日志
- error.log: 错误日志
- audit.log: 审计日志
- access.log: 访问日志
