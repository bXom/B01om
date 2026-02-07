# B01om 项目文档

## 1. 项目概述

B01om 是一个基于 Spring-Cloud 的微服务架构平台，提供以用户管理、权限控制、主体管理为基础功能的企业级应用。

## 2. 项目架构

### 2.1 模块结构

```
b01om/
├── common          # 公共模块：DTO、工具类、异常处理
├── config          # 配置中心：Spring Cloud Config Server
├── eureka          # 服务注册中心：Eureka Server
├── gateway         # API 网关：Spring Cloud Gateway
├── mysql-service   # MySQL 数据服务：数据持久化层
├── user-service    # 用户服务：认证、权限计算
└── web-api         # Web API：对外暴露的 REST 接口
```

### 2.2 服务依赖关系

```
                    ┌─────────────┐
                    │   Gateway   │
                    └──────┬──────┘
                           ▼
             ┌─────────────────────────┐
             ▼                         ▼
        ┌──────────┐              ┌──────────┐
        │    Web   │              │   User   │
        │    API   │              │  Service │
        └────┬─────┘              └────┬─────┘
             │                         │
             └─────────────────────────┘
                           │ 
                           ▼
                   ┌─────────────┐
                   │   MySQL     │
                   │   Service   │
                   └─────────────┘
```

### 2.3 MVC架构

- web-api 模块和 user-service 模块分层架构：
    
    | 层级 | 职责 | 示例 |
    |------|------|------|
    | Controller | 接收请求、参数校验、调用 Service | `UserController` |
    | Service | 业务逻辑 | `UserServiceImpl` |
    | DTO | 数据传输对象 | `UserDTO`, `UserAddDTO` |

- mysql-service 模块分层架构：

  | 层级 | 职责 | 示例 |
  |------|------|------|
  | Controller | 接收请求、参数校验、调用 Service | `UserController` |
  | Service | 业务逻辑、DTO 转换 | `UserServiceImpl` |
  | Mapper | 数据库操作 (MyBatis Plus) | `UserMapper` |
  | Model/Entity | 数据库实体 | `User` |

## 3. 技术栈约束

### 3.1 核心技术栈

- GroupId：com.bxom
- ArtifactId：b01om
- java版本是向下兼容的1.8+
- Spring Boot 2.7.5 
- Spring Cloud 2021.0.4 
- 数据库: MySQL 8.0 
- 数据库连接池: Druid 1.2.6 
- 安全认证: Auth0 JWT 3.18.2 
- ORM 框架: MyBatis Plus 3.5.3.1

### 3.2 辅助技术栈

- 服务注册与发现：Spring Cloud Eureka
- 服务间调用：Spring Cloud OpenFeign
- API 网关：Spring Cloud Gateway
- 配置中心：Spring Cloud Config
- 认证令牌：Auth0 JWT 3.18.2
- 代码简化：Lombok 1.18.24
- 工具类库：Hutool 5.8.11
- API 文档：Knife4j 3.0.3

### 3.3 版本约束

- **JDK**: 必须使用 Java 1.8，不得升级到更高版本
- **Spring Boot**: 锁定 2.7.x 系列
- **Spring Cloud**: 使用 2021.0.x (Jubilee) 版本线

## 4. 设计原则

### 4.1 DTO 分离原则

数据库 Entity 仅在 `mysql-service` 模块内部使用，其他服务通过 DTO 进行数据交换。

```
┌─────────────────────────────────────────────────────────────┐
│                            common                           │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  UserDTO, UserAddDTO, UserUpdateDTO, UserQueryDTO   │    │
│  │  EFRDTO, UFRDTO, RFRDTO, UPRDTO ...                 │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                              ▲
                              │ 引用
              ┌───────────────┼───────────────┐
              │               │               │
┌─────────────┴───┐   ┌───────┴───────┐   ┌───┴─────────────┐
│  user-service   │   │    web-api    │   │  mysql-service  │
│     (DTO)       │   │    (DTO)      │   │  (Entity + DTO) │
└─────────────────┘   └───────────────┘   └─────────────────┘
```

### 4.2 DTO 命名规范

| 后缀 | 用途 | 示例 |
|------|------|------|
| `DTO` | 响应数据 | `UserDTO` |
| `AddDTO` | 新增请求 | `UserAddDTO` |
| `UpdateDTO` | 更新请求 | `UserUpdateDTO` |
| `QueryDTO` | 查询条件 | `UserQueryDTO` |

### 4.3 RESTful API 规范

```
GET    /users              # 分页查询
GET    /users/list         # 列表查询
GET    /users/{uuid}       # 根据 ID 查询
POST   /users              # 新增
PUT    /users/{uuid}       # 更新
DELETE /users/{uuid}       # 删除
DELETE /users              # 批量删除 (Body: uuid 列表)
```

### 4.4 时间类型规范

| 位置 | 类型 | 格式化 |
|------|------|--------|
| Entity (mysql-service) | `LocalDateTime` | - |
| DTO (common) | `Date` | `@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")` |

使用 `DtoConverter` 工具类进行转换：
```java
public static Date toDate(LocalDateTime localDateTime) {
    if (localDateTime == null) return null;
    return Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
}
```

## 5. 编码风格

### 5.1 包结构

```
com.bxom.b01om.{module}
├── config          # 配置类
├── controller      # 控制器
├── service         # 服务接口
│   └── impl        # 服务实现
├── feign           # Feign 客户端 (user-service/web-api)
├── mapper          # MyBatis Mapper (mysql-service)
├── model           # Entity 实体 (common/mysql-service)
│   ├── converter   # DTO 转换器 (mysql-service)
│   └── dto         # Entity DTO (common)
└── mapper          # MyBatis Mapper (mysql-service)
```

### 5.2 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 类名 | 大驼峰 | `UserService` |
| 方法名 | 小驼峰 | `searchPage` |
| 常量 | 全大写下划线 | `DEFAULT_PASSWORD` |
| 数据库表 | 小写下划线 | `user_role` |
| 数据库字段 | 小写下划线 | `entity_uuid` |

### 5.3 Service 方法命名

| 操作 | 方法名 | 返回类型 |
|------|--------|----------|
| 分页查询 | `searchPage` | `IPage<DTO>` |
| 列表查询 | `searchList` | `List<DTO>` |
| 单个查询 | `getByUuid` | `DTO` |
| 新增 | `add` | `DTO` |
| 批量新增 | `addList` | `boolean` |
| 更新 | `updateByUuid` | `boolean` |
| 删除 | `deleteByUuidList` | `boolean` |

### 5.4 注解使用

```java
// Controller
@Api(tags = "用户管理接口")
@RestController
@RequestMapping("/users")

// 方法
@ApiOperation("分页查询用户")
@GetMapping

// 参数
@ApiParam("页码") @RequestParam(defaultValue = "1") Long current
@ApiParam("用户UUID") @PathVariable String uuid
@RequestBody UserAddDTO addDTO
```

## 6. 业务逻辑

### 6.1 认证流程

```
┌──────────┐      ┌───────────────┐      ┌───────────────┐
│  Client  │─────▶│ user-service  │─────▶│ mysql-service │
│          │      │  /auth/login  │      │  /users/login │
└──────────┘      └───────┬───────┘      └───────────────┘
                          │
                          ▼
                  ┌───────────────┐
                  │  计算用户权限   │
                  │   生成 JWT     │
                  └───────────────┘
                          │
                          ▼
              ┌─────────────────────┐
              │  返回 LoginResponse │
              │  - token            │
              │  - permissions      │
              └─────────────────────┘
```

### 6.2 密码处理

- 密码使用 MD5 加密存储
- 默认密码: `12345pass` (MD5: ``)
- 密码在客户端加密后传输

### 6.3 实体公共字段

1. 唯一码字段：
   - `uuid`: 唯一码，主键
2. 状态字段：
   - `status = "01"`: 有效；`status = "00"`: 无效（软删除）
   - `invalid_time`: 无效时间
3. 时间字段：
   - `insert_time`: 创建时间
   - `update_time`: 最后修改时间

## 7. 权限管理

### 7.1 权限模型 (Entity-User-Func)

```
                    ┌─────────────┐
                    │   Entity    │  主体（企业/个人）
                    │   (主体)     │
                    └──────┬──────┘
                           │
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
    ┌────────────┐  ┌────────────┐  ┌────────────┐
    │   User     │  │   Role     │  │  FuncInfo  │
    │   (用户)    │  │   (角色)   │  │   (方法)    │
    └────────────┘  └────────────┘  └────────────┘
```

### 7.2 权限计算公式

```
用户权限 = 主体默认方法 ∪ 用户直接授权方法 ∪ 用户角色方法

其中:
- 主体默认方法 = entity_func_relate 中 funcType 为 EntityDefault 或 UserDefault 的方法
- 用户直接授权方法 = user_func_relate ∩ 主体方法
- 用户角色方法 = (user_role → role_func_relate) ∩ 主体方法
```

### 7.3 方法类型 (funcType)

| 类型 | 说明 | 授权范围 |
|------|------|----------|
| `EntityDefault` | 主体默认方法 | 主体下所有用户自动拥有 |
| `UserDefault` | 用户默认方法 | 用户自动拥有的基础权限 |
| `EntityFeat` | 主体特性方法 | 需单独授权的主体功能 |
| `Sys` | 系统方法 | 系统管理员专用 |

### 7.4 权限关联表

| 关联表 | 说明 | 关联关系 |
|--------|------|----------|
| `entity_func_relate` | 主体-方法 | 主体拥有哪些方法 |
| `user_func_relate` | 用户-方法 | 用户直接拥有的方法 |
| `user_role` | 用户-角色 | 用户拥有的角色 |
| `role_func_relate` | 角色-方法 | 角色拥有的方法 |
| `entity_per_relate` | 主体-主体-方法 | 主体间授权 |

### 7.5 权限检查

```java
// Controller 中进行权限检查
private void checkPermission(String operatorUuid, String entityUuid, String funcName) {
    if (!permissionService.hasPermission(operatorUuid, entityUuid, funcName)) {
        throw new BusinessException(ResultCode.FORBIDDEN, "权限不足");
    }
}
```

## 8. 数据结构

### 8.1 核心实体

#### BaseModel (公共字段)

| 字段 | 类型 | 说明 |
|------|------|------|
| uuid | VARCHAR(64) | 主键 (UUID) |
| insert_time | DATETIME | 创建时间 |
| update_time | DATETIME | 更新时间 |
| status | ENUM('00','01') | 状态 |
| invalid_time | DATETIME | 失效时间 |

#### User (用户表)

| 字段 | 类型 | 说明 |
|------|------|------|
| phone | VARCHAR(18) | 手机号 (唯一) |
| name | VARCHAR(64) | 真实姓名 |
| username | VARCHAR(255) | 登录名 (唯一) |
| email | VARCHAR(255) | 邮箱 |
| password | VARCHAR(255) | 密码 (MD5) |
| token | VARCHAR(1024) | 登录 Token |
| ip | VARCHAR(15) | 最后登录 IP |
| last_login_time | DATETIME | 最后登录时间 |
| entity_uuid | VARCHAR(64) | 所属主体 |

#### Entity (主体表)

| 字段 | 类型 | 说明 |
|------|------|------|
| name | VARCHAR(255) | 主体名称 (唯一) |
| type | ENUM('00','01') | 类型 (00自然人/01企业法人) |
| code | VARCHAR(18) | 身份证/统一信用代码 (唯一) |
| region_code | VARCHAR(16) | 行政区划代码 |
| address | VARCHAR(255) | 详细地址 |
| legal_represent | VARCHAR(255) | 法定代表人 |
| phone | VARCHAR(128) | 联系电话 |
| email | VARCHAR(255) | 邮箱 |
| verified | ENUM('00','01','02') | 认证状态 |
| verified_time | DATETIME | 认证时间 |

#### FuncInfo (方法注册表)

| 字段 | 类型 | 说明 |
|------|------|------|
| func_name | VARCHAR(255) | 方法名 |
| func_type | ENUM | 方法类型 |
| func_note | VARCHAR(255) | 方法描述 |

#### Role (角色表)

| 字段 | 类型 | 说明 |
|------|------|------|
| name_key | VARCHAR(64) | 角色英文名 (唯一) |
| name_val | VARCHAR(64) | 角色中文名 (唯一) |

### 8.2 关联表

| 表名 | 关联字段 | 说明 |
|------|----------|------|
| user_role | user_uuid, role_uuid | 用户-角色 |
| entity_func_relate | entity_uuid, func_uuid | 主体-方法 |
| user_func_relate | user_uuid, func_uuid | 用户-方法 |
| role_func_relate | role_uuid, func_uuid | 角色-方法 |
| entity_per_relate | ori_uuid, target_uuid, func_uuid | 主体间授权 |

### 8.3 DTO 缩写对照

| 缩写 | 全称 | 说明 |
|------|------|------|
| UPR | User Permission Relate | 用户-角色关联 (复用为用户权限) |
| EFR | Entity Func Relate | 主体-方法关联 |
| UFR | User Func Relate | 用户-方法关联 |
| RFR | Role Func Relate | 角色-方法关联 |
| EPR | Entity Per Relate | 主体间授权 |

### 8.4 实体关系图

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│    Entity    │◀──────│     User     │───────│     Role     │
│   (主体)      │  1:N  │    (用户)    │  N:M  │    (角色)     │
└──────┬───────┘       └──────┬───────┘       └──────┬───────┘
       │                      │                      │
       │ N:M                  │ N:M                  │ N:M
       ▼                      ▼                      ▼
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│entity_func   │       │ user_func    │       │ role_func    │
│  _relate     │       │  _relate     │       │  _relate     │
└──────┬───────┘       └──────┬───────┘       └──────┬───────┘
       │                      │                      │
       └──────────────────────┼──────────────────────┘
                              ▼
                       ┌──────────────┐
                       │   FuncInfo   │
                       │    (方法)     │
                       └──────────────┘
```

## 9. 服务端口规划

| 服务 | 端口 | 说明 |
|------|------|------|
| b01om-eureka | 8761 | 服务注册中心 |
| b01om-config | 8888 | 配置中心 |
| b01om-gateway | 8080 | API 网关 |
| b01om-mysql-service | 8081 | MySQL 数据服务 |
| b01om-user-service | 8082 | 用户服务 |
| b01om-web-api | 8083 | Web API |

## 10. 开发指南

### 10.1 新增实体流程

1. 在 `mysql-service` 创建 Entity 类
2. 在 `common` 模块创建对应 DTO (DTO, AddDTO, UpdateDTO, QueryDTO)
3. 在 `mysql-service` 创建 Mapper, Service, Controller
4. 在 `mysql-service/dto/converter/DtoConverter` 添加转换方法

### 10.2 新增 API 流程

1. 在 `mysql-service` Controller 添加端点
2. 在调用方 (如 user-service) 的 Feign Client 添加方法声明
3. 在调用方 Service 中使用 Feign Client

### 10.3 权限配置流程

1. 在 `func_info` 表注册新方法
2. 通过 `entity_func_relate` 将方法授权给主体
3. 通过 `role_func_relate` 或 `user_func_relate` 授权给角色/用户
