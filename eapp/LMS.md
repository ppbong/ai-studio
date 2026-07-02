# LMS 学习管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 LMS（Learning Management System）学习管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
LMS 系统作为企业级应用体系的学习管理平台，负责课程管理、学习计划、在线学习、考试评估等培训业务，与 HR、OA 系统紧密协同，提升员工能力和组织绩效。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 课程管理 | 课程创建、课程分类、课程上架 | 高 |
| 2 | 学习计划 | 学习路径、学习任务、学习跟踪 | 高 |
| 3 | 在线学习 | 视频学习、文档学习、互动学习 | 高 |
| 4 | 考试评估 | 在线考试、测验、成绩管理 | 高 |
| 5 | 学习统计 | 学习时长、完成率、学习报告 | 高 |
| 6 | 培训管理 | 培训计划、培训记录、证书管理 | 中 |
| 7 | 知识库 | 知识文档、问答社区、专家咨询 | 中 |

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
| 课程模块 | 课程管理 | 课程创建、分类、上架 |
| 学习模块 | 在线学习 | 视频、文档学习 |
| 考试模块 | 考试评估 | 在线考试、测验 |
| 计划模块 | 学习计划 | 学习路径、任务 |
| 统计模块 | 学习统计 | 时长、完成率 |
| 培训模块 | 培训管理 | 培训计划、证书 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/lms/
  │   │   │   ├── controller/
  │   │   │   │   ├── CourseController.java      # 课程管理
  │   │   │   │   ├── LearningController.java    # 学习管理
  │   │   │   │   ├── ExamController.java        # 考试管理
  │   │   │   │   ├── PlanController.java        # 学习计划
  │   │   │   │   └── StatisticsController.java  # 学习统计
  │   │   │   ├── service/
  │   │   │   │   ├── CourseService.java
  │   │   │   │   ├── LearningService.java
  │   │   │   │   ├── ExamService.java
  │   │   │   │   ├── PlanService.java
  │   │   │   │   └── StatisticsService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── LmsApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── player/                           # 播放器组件
  │   │   │   └── VideoPlayer.vue
  │   ├── views/
  │   │   ├── course/                           # 课程管理
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── learning/                         # 在线学习
  │   │   │   └── index.vue
  │   │   ├── exam/                             # 考试管理
  │   │   │   └── index.vue
  │   │   ├── plan/                             # 学习计划
  │   │   │   └── index.vue
  │   │   └── statistics/                       # 学习统计
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 CourseService (课程服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createCourse` | 创建课程 | `CourseCreateRequest request` | `CourseResponse` | 抛出`BusinessException` |
| `publishCourse` | 发布课程 | `Long courseId` | `CourseResponse` | 抛出`CourseNotFoundException` |
| `getCourses` | 查询课程列表 | `CourseSearchRequest request` | `Page<CourseResponse>` | - |

#### 5.1.2 LearningService (学习服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `startLearning` | 开始学习 | `Long courseId, Long userId` | `LearningRecordResponse` | 抛出`CourseNotFoundException` |
| `completeLearning` | 完成学习 | `Long recordId` | `LearningRecordResponse` | 抛出`RecordNotFoundException` |
| `getLearningProgress` | 获取学习进度 | `Long courseId, Long userId` | `ProgressResponse` | - |

#### 5.1.3 ExamService (考试服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createExam` | 创建考试 | `ExamCreateRequest request` | `ExamResponse` | 抛出`BusinessException` |
| `startExam` | 开始考试 | `Long examId, Long userId` | `ExamRecordResponse` | 抛出`ExamNotFoundException` |
| `submitExam` | 提交考试 | `Long recordId, List<AnswerRequest> answers` | `ExamResultResponse` | 抛出`RecordNotFoundException` |

### 5.2 DTO 结构定义

**CourseCreateRequest（创建课程请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| courseName | String | 课程名称 | 非空 |
| categoryId | Long | 分类 ID | 非空 |
| description | String | 课程描述 | 可选 |
| duration | Integer | 时长(分钟) | 非空 |
| difficulty | String | 难度级别 | 非空 |
| coverUrl | String | 封面图片 | 可选 |

**LearningRecordResponse（学习记录响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| id | Long | 记录 ID |
| courseId | Long | 课程 ID |
| courseName | String | 课程名称 |
| userId | Long | 用户 ID |
| progress | BigDecimal | 学习进度 |
| status | String | 学习状态 |
| startTime | LocalDateTime | 开始时间 |
| endTime | LocalDateTime | 结束时间 |

**ExamResultResponse（考试结果响应）**
| 字段名 | 类型 | 含义 |
| --- | --- | --- |
| recordId | Long | 考试记录 ID |
| examId | Long | 考试 ID |
| score | Integer | 得分 |
| totalScore | Integer | 总分 |
| passed | Boolean | 是否通过 |
| completedAt | LocalDateTime | 完成时间 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 课程表 (lms_course)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 课程 ID |
| course_name | VARCHAR(200) | NOT NULL | 课程名称 |
| category_id | BIGINT | NOT NULL | 分类 ID |
| description | TEXT | - | 课程描述 |
| duration | INT | NOT NULL | 时长(分钟) |
| difficulty | VARCHAR(20) | NOT NULL | 难度级别 |
| cover_url | VARCHAR(500) | - | 封面图片 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_by | BIGINT | NOT NULL | 创建人 ID |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 课程章节表 (lms_course_chapter)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 章节 ID |
| course_id | BIGINT | FOREIGN KEY, NOT NULL | 课程 ID |
| chapter_name | VARCHAR(200) | NOT NULL | 章节名称 |
| content_type | VARCHAR(20) | NOT NULL | 内容类型 |
| content_url | VARCHAR(500) | NOT NULL | 内容地址 |
| duration | INT | - | 时长(秒) |
| sort_order | INT | DEFAULT 0 | 排序 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 学习记录表 (lms_learning_record)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 记录 ID |
| course_id | BIGINT | FOREIGN KEY, NOT NULL | 课程 ID |
| user_id | BIGINT | NOT NULL | 用户 ID |
| progress | DECIMAL(5,2) | DEFAULT 0 | 学习进度 |
| status | VARCHAR(20) | NOT NULL | 学习状态 |
| start_time | DATETIME | NOT NULL | 开始时间 |
| end_time | DATETIME | - | 结束时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.4 考试表 (lms_exam)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 考试 ID |
| exam_name | VARCHAR(200) | NOT NULL | 考试名称 |
| course_id | BIGINT | FOREIGN KEY | 关联课程 ID |
| duration | INT | NOT NULL | 时长(分钟) |
| pass_score | INT | NOT NULL | 及格分数 |
| total_score | INT | NOT NULL | 总分 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 考试记录表 (lms_exam_record)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 记录 ID |
| exam_id | BIGINT | FOREIGN KEY, NOT NULL | 考试 ID |
| user_id | BIGINT | NOT NULL | 用户 ID |
| score | INT | NOT NULL | 得分 |
| passed | BOOLEAN | NOT NULL | 是否通过 |
| started_at | DATETIME | NOT NULL | 开始时间 |
| completed_at | DATETIME | - | 完成时间 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 课程管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/lms/courses` | GET | CourseController.java | 查询课程列表 |
| `/api/lms/courses/{id}` | GET | CourseController.java | 查询课程详情 |
| `/api/lms/courses` | POST | CourseController.java | 创建课程 |
| `/api/lms/courses/{id}/publish` | POST | CourseController.java | 发布课程 |

### 7.2 学习管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/lms/learning/start` | POST | LearningController.java | 开始学习 |
| `/api/lms/learning/progress` | GET | LearningController.java | 获取学习进度 |
| `/api/lms/learning/complete` | POST | LearningController.java | 完成学习 |

### 7.3 考试管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/lms/exams` | GET | ExamController.java | 查询考试列表 |
| `/api/lms/exams/{id}/start` | POST | ExamController.java | 开始考试 |
| `/api/lms/exams/{id}/submit` | POST | ExamController.java | 提交考试 |

### 7.4 学习统计接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/lms/statistics/user/{userId}` | GET | StatisticsController.java | 用户学习统计 |
| `/api/lms/statistics/course/{courseId}` | GET | StatisticsController.java | 课程学习统计 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 课程管理 | course:read, course:write | 查看和创建课程 |
| 学习管理 | learning:read, learning:write | 学习和记录 |
| 考试管理 | exam:read, exam:write | 查看和参与考试 |

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
| HR | REST API | 员工培训记录同步 |
| OA | REST API | 培训审批流程 |

---

**文档结束**