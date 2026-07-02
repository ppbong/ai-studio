# HR 人力资源管理系统设计文档

## 1. 文档概述

### 1.1 文档目的
本文档详细描述 HR（Human Resources）人力资源管理系统的设计方案，包括系统架构、功能模块、API 接口、数据模型等，为系统开发和部署提供技术依据。

### 1.2 系统定位
HR 系统作为企业级应用体系的人力资源管理平台，负责组织架构、员工管理、招聘管理、培训发展、绩效考核、薪酬福利等人力资源管理业务，与 ERP、OA 系统紧密协同。

### 1.3 文档版本
| 版本 | 日期 | 作者 | 变更说明 |
| --- | --- | --- | --- |
| V1.0 | 2026-06-03 | 架构组 | 初始版本 |

---

## 2. 需求分析

### 2.1 功能需求

| 序号 | 需求点 | 需求描述 | 优先级 |
| --- | --- | --- | --- |
| 1 | 组织架构 | 部门管理、岗位管理、编制管理 | 高 |
| 2 | 员工管理 | 员工档案、入职离职、转正调岗 | 高 |
| 3 | 招聘管理 | 招聘需求、简历管理、面试管理 | 高 |
| 4 | 培训发展 | 培训计划、培训课程、培训记录 | 中 |
| 5 | 绩效考核 | 考核方案、考核执行、考核结果 | 高 |
| 6 | 薪酬福利 | 薪资核算、社保公积金、福利管理 | 高 |
| 7 | 考勤管理 | 考勤规则、考勤统计、请假加班 | 高 |

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
| 组织模块 | 组织架构管理 | 部门、岗位、编制 |
| 员工模块 | 员工管理 | 员工档案、异动管理 |
| 招聘模块 | 招聘管理 | 招聘需求、面试 |
| 培训模块 | 培训发展 | 培训计划、课程 |
| 绩效模块 | 绩效考核 | 考核方案、执行 |
| 薪酬模块 | 薪酬福利 | 薪资核算、社保 |
| 考勤模块 | 考勤管理 | 考勤规则、统计 |

---

## 4. 目录结构

```plaintext
backend/
  ├── src/
  │   ├── main/
  │   │   ├── java/com/example/hr/
  │   │   │   ├── controller/
  │   │   │   │   ├── OrganizationController.java # 组织架构
  │   │   │   │   ├── EmployeeController.java     # 员工管理
  │   │   │   │   ├── RecruitmentController.java  # 招聘管理
  │   │   │   │   ├── TrainingController.java     # 培训发展
  │   │   │   │   ├── PerformanceController.java  # 绩效考核
  │   │   │   │   ├── PayrollController.java      # 薪酬福利
  │   │   │   │   └── AttendanceController.java   # 考勤管理
  │   │   │   ├── service/
  │   │   │   │   ├── OrganizationService.java
  │   │   │   │   ├── EmployeeService.java
  │   │   │   │   ├── RecruitmentService.java
  │   │   │   │   ├── TrainingService.java
  │   │   │   │   ├── PerformanceService.java
  │   │   │   │   ├── PayrollService.java
  │   │   │   │   └── AttendanceService.java
  │   │   │   ├── repository/
  │   │   │   ├── entity/
  │   │   │   ├── dto/
  │   │   │   ├── config/
  │   │   │   └── HrApplication.java
  │   └── resources/
  │       ├── application.yml
  │       └── schema.sql
  └── pom.xml

frontend/
  ├── src/
  │   ├── components/
  │   │   ├── org/                               # 组织架构组件
  │   │   │   └── OrgTree.vue
  │   ├── views/
  │   │   ├── organization/                      # 组织架构
  │   │   │   └── index.vue
  │   │   ├── employee/                          # 员工管理
  │   │   │   ├── list.vue
  │   │   │   └── detail.vue
  │   │   ├── recruitment/                       # 招聘管理
  │   │   │   └── index.vue
  │   │   ├── performance/                       # 绩效考核
  │   │   │   └── index.vue
  │   │   ├── payroll/                           # 薪酬福利
  │   │   │   └── index.vue
  │   │   └── attendance/                        # 考勤管理
  │   │       └── index.vue
  │   ├── api/
  │   ├── store/
  │   └── main.ts
  └── package.json
```

---

## 5. 关键类与方法设计

### 5.1 核心服务类

#### 5.1.1 EmployeeService (员工服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createEmployee` | 创建员工 | `EmployeeCreateRequest request` | `EmployeeResponse` | 抛出`BusinessException` |
| `updateEmployee` | 更新员工 | `Long employeeId, EmployeeUpdateRequest request` | `EmployeeResponse` | 抛出`EmployeeNotFoundException` |
| `transferEmployee` | 员工调动 | `Long employeeId, TransferRequest request` | `EmployeeResponse` | 抛出`EmployeeNotFoundException` |
| `resignEmployee` | 员工离职 | `Long employeeId, ResignRequest request` | `EmployeeResponse` | 抛出`EmployeeNotFoundException` |
| `getEmployees` | 查询员工列表 | `EmployeeSearchRequest request` | `Page<EmployeeResponse>` | - |

#### 5.1.2 PayrollService (薪酬服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `calculatePayroll` | 核算薪资 | `PayrollCalculateRequest request` | `PayrollResponse` | 抛出`BusinessException` |
| `generatePayroll` | 生成工资单 | `Long employeeId, Integer month` | `PayslipResponse` | 抛出`EmployeeNotFoundException` |
| `getPayrollRecords` | 查询薪资记录 | `PayrollSearchRequest request` | `Page<PayrollResponse>` | - |

#### 5.1.3 PerformanceService (绩效服务)

| 方法名 | 功能说明 | 参数 | 返回值 | 失败返回 |
| --- | --- | --- | --- | --- |
| `createPerformancePlan` | 创建考核方案 | `PerformancePlanCreateRequest request` | `PerformancePlanResponse` | 抛出`BusinessException` |
| `executePerformance` | 执行考核 | `Long planId, PerformanceExecuteRequest request` | `PerformanceResultResponse` | 抛出`PlanNotFoundException` |
| `getPerformanceResults` | 查询考核结果 | `PerformanceSearchRequest request` | `Page<PerformanceResultResponse>` | - |

### 5.2 DTO 结构定义

**EmployeeCreateRequest（创建员工请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| employeeNo | String | 员工编号 | 非空，唯一 |
| name | String | 姓名 | 非空 |
| gender | String | 性别 | 非空 |
| departmentId | Long | 部门 ID | 非空 |
| positionId | Long | 岗位 ID | 非空 |
| hireDate | LocalDate | 入职日期 | 非空 |
| email | String | 邮箱 | 可选 |
| phone | String | 电话 | 可选 |

**PayrollItemRequest（薪资项请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| itemType | String | 薪资项类型 | 非空 |
| amount | BigDecimal | 金额 | 非空 |
| remark | String | 备注 | 可选 |

**PerformancePlanCreateRequest（创建考核方案请求）**
| 字段名 | 类型 | 含义 | 约束 |
| --- | --- | --- | --- |
| planName | String | 方案名称 | 非空 |
| cycleType | String | 考核周期 | 非空 |
| startDate | LocalDate | 开始日期 | 非空 |
| endDate | LocalDate | 结束日期 | 非空 |
| indicators | List<PerformanceIndicator> | 考核指标 | 非空 |

---

## 6. 数据库与数据结构设计

### 6.1 数据库表设计

#### 6.1.1 部门表 (hr_department)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 部门 ID |
| code | VARCHAR(50) | UNIQUE, NOT NULL | 部门编码 |
| name | VARCHAR(100) | NOT NULL | 部门名称 |
| parent_id | BIGINT | FOREIGN KEY | 父部门 ID |
| manager_id | BIGINT | - | 部门负责人 ID |
| level | INT | NOT NULL | 层级 |
| status | VARCHAR(20) | DEFAULT 'ACTIVE' | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.2 岗位表 (hr_position)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 岗位 ID |
| code | VARCHAR(50) | UNIQUE, NOT NULL | 岗位编码 |
| name | VARCHAR(100) | NOT NULL | 岗位名称 |
| department_id | BIGINT | FOREIGN KEY, NOT NULL | 所属部门 |
| level | VARCHAR(50) | - | 职级 |
| headcount | INT | DEFAULT 0 | 编制人数 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.3 员工表 (hr_employee)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 员工 ID |
| employee_no | VARCHAR(50) | UNIQUE, NOT NULL | 员工编号 |
| name | VARCHAR(100) | NOT NULL | 姓名 |
| gender | VARCHAR(10) | NOT NULL | 性别 |
| department_id | BIGINT | FOREIGN KEY, NOT NULL | 部门 ID |
| position_id | BIGINT | FOREIGN KEY, NOT NULL | 岗位 ID |
| hire_date | DATE | NOT NULL | 入职日期 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| email | VARCHAR(100) | - | 邮箱 |
| phone | VARCHAR(20) | - | 电话 |
| created_at | DATETIME | NOT NULL | 创建时间 |
| updated_at | DATETIME | NOT NULL | 更新时间 |

#### 6.1.4 员工异动表 (hr_employee_transfer)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 异动 ID |
| employee_id | BIGINT | FOREIGN KEY, NOT NULL | 员工 ID |
| transfer_type | VARCHAR(50) | NOT NULL | 异动类型 |
| from_department_id | BIGINT | - | 原部门 ID |
| to_department_id | BIGINT | - | 新部门 ID |
| from_position_id | BIGINT | - | 原岗位 ID |
| to_position_id | BIGINT | - | 新岗位 ID |
| effective_date | DATE | NOT NULL | 生效日期 |
| reason | VARCHAR(500) | - | 异动原因 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.5 薪资记录表 (hr_payroll)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 薪资 ID |
| employee_id | BIGINT | FOREIGN KEY, NOT NULL | 员工 ID |
| month | INT | NOT NULL | 月份 |
| year | INT | NOT NULL | 年份 |
| base_salary | DECIMAL(12,2) | NOT NULL | 基本工资 |
| performance_bonus | DECIMAL(12,2) | DEFAULT 0 | 绩效奖金 |
| allowance | DECIMAL(12,2) | DEFAULT 0 | 补贴 |
| deduction | DECIMAL(12,2) | DEFAULT 0 | 扣款 |
| social_security | DECIMAL(12,2) | DEFAULT 0 | 社保 |
| housing_fund | DECIMAL(12,2) | DEFAULT 0 | 公积金 |
| tax | DECIMAL(12,2) | DEFAULT 0 | 个税 |
| net_salary | DECIMAL(12,2) | NOT NULL | 实发工资 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.6 绩效考核表 (hr_performance)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 考核 ID |
| plan_id | BIGINT | FOREIGN KEY, NOT NULL | 方案 ID |
| employee_id | BIGINT | FOREIGN KEY, NOT NULL | 员工 ID |
| cycle_start | DATE | NOT NULL | 考核周期开始 |
| cycle_end | DATE | NOT NULL | 考核周期结束 |
| total_score | DECIMAL(5,2) | - | 总分 |
| rating | VARCHAR(20) | - | 等级 |
| comment | VARCHAR(500) | - | 评语 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| created_at | DATETIME | NOT NULL | 创建时间 |

#### 6.1.7 考勤记录表 (hr_attendance)

| 字段名 | 类型 | 约束 | 说明 |
| --- | --- | --- | --- |
| id | BIGINT | PRIMARY KEY, AUTO_INCREMENT | 考勤 ID |
| employee_id | BIGINT | FOREIGN KEY, NOT NULL | 员工 ID |
| date | DATE | NOT NULL | 日期 |
| check_in_time | TIME | - | 上班时间 |
| check_out_time | TIME | - | 下班时间 |
| status | VARCHAR(20) | NOT NULL | 状态 |
| work_hours | DECIMAL(5,2) | - | 工作时长 |
| overtime_hours | DECIMAL(5,2) | DEFAULT 0 | 加班时长 |
| leave_hours | DECIMAL(5,2) | DEFAULT 0 | 请假时长 |
| created_at | DATETIME | NOT NULL | 创建时间 |

---

## 7. API 接口设计

### 7.1 组织架构接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/hr/departments` | GET | OrganizationController.java | 查询部门列表 |
| `/api/hr/departments/tree` | GET | OrganizationController.java | 查询组织树 |
| `/api/hr/departments` | POST | OrganizationController.java | 创建部门 |
| `/api/hr/positions` | GET | OrganizationController.java | 查询岗位列表 |

### 7.2 员工管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/hr/employees` | GET | EmployeeController.java | 查询员工列表 |
| `/api/hr/employees/{id}` | GET | EmployeeController.java | 查询员工详情 |
| `/api/hr/employees` | POST | EmployeeController.java | 创建员工 |
| `/api/hr/employees/{id}/transfer` | POST | EmployeeController.java | 员工调动 |
| `/api/hr/employees/{id}/resign` | POST | EmployeeController.java | 员工离职 |

### 7.3 薪酬福利接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/hr/payroll` | GET | PayrollController.java | 查询薪资记录 |
| `/api/hr/payroll/calculate` | POST | PayrollController.java | 核算薪资 |
| `/api/hr/payroll/{id}/payslip` | GET | PayrollController.java | 获取工资单 |

### 7.4 绩效考核接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/hr/performance/plans` | GET | PerformanceController.java | 查询考核方案 |
| `/api/hr/performance/plans` | POST | PerformanceController.java | 创建考核方案 |
| `/api/hr/performance/results` | GET | PerformanceController.java | 查询考核结果 |

### 7.5 考勤管理接口

| API 路径 | HTTP 方法 | Controller 文件 | 功能描述 |
| --- | --- | --- | --- |
| `/api/hr/attendance` | GET | AttendanceController.java | 查询考勤记录 |
| `/api/hr/attendance/statistics` | GET | AttendanceController.java | 考勤统计 |

---

## 8. 安全设计

### 8.1 认证机制
- 通过 SSO 系统进行统一身份认证
- 使用 JWT 令牌进行接口访问控制

### 8.2 权限控制

| 资源 | 权限 | 说明 |
| --- | --- | --- |
| 员工管理 | employee:read, employee:write | 查看和编辑员工 |
| 薪酬管理 | payroll:read, payroll:write | 查看和核算薪资 |
| 绩效考核 | performance:read, performance:write | 查看和执行考核 |

---

## 9. 部署与集成方案

### 9.1 依赖与环境

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Spring Boot | 3.2.x | 后端框架 |
| Spring Security | 6.2.x | 安全框架 |
| PostgreSQL | 15+ | 数据库 |
| Redis | 7.0+ | 缓存 |

### 9.2 与其他系统集成

| 系统 | 集成方式 | 说明 |
| --- | --- | --- |
| SSO | OAuth2.0 | 统一身份认证 |
| ERP | REST API + 消息队列 | 薪资数据同步 |
| OA | REST API | 考勤数据、请假审批 |

---

**文档结束**