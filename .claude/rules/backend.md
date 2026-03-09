---
paths:
  - "server/**/*.go"
---

### 目录结构

```text
server/
├── internal/                  # 核心业务代码（禁止外部包直接引用）
│   ├── controller/            # HTTP 入口：参数绑定、校验、调用 Service、返回统一响应、编写 OpenAPI 注释
│   │   └── user.go            # 禁止：直接操作 DB/Repo、返回 model.*、处理数据库错误语义
│   ├── service/               # 业务编排、事务控制、调用 Repo/Converter/Clients、返回 dto.*
│   │   └── user.go            # 禁止：依赖 gin.Context、返回 model.*、绕过 Repo 访问数据库
│   ├── repo/                  # MySQL/Redis 数据访问，返回 model.*，接收 tx 执行事务
│   │   └── user.go            # 禁止：写业务逻辑、暴露 DB()、直接整体更新请求 DTO
│   ├── converter/             # model.* 与 dto.* 的转换、字段裁剪与脱敏
│   │   └── user.go            # 禁止：访问数据库、写业务逻辑
│   ├── model/                 # 数据库实体定义，仅用于持久化映射，禁止添加 json 标签
│   │   └── user.go            # 禁止：放 request/response/query 结构
│   ├── dto/                   # 请求/响应结构体，推荐按模块拆分 request.go / response.go
│   │   └── user/
│   │       ├── request.go     # 请求 DTO：允许 json/form/binding，禁止 gorm
│   │       └── response.go    # 响应 DTO：允许 json，禁止 gorm/binding
│   ├── middleware/            # Gin 中间件
│   │   ├── jwt.go             # JWT 鉴权（JWTAuth()）
│   │   └── requestid.go       # TraceID 注入（生成或从 X-Trace-Id Header 读取）
│   ├── router/                # 路由注册；按模块拆分，router.go 只负责聚合注册
│   │   ├── router.go          # 聚合注册所有路由
│   │   └── user.go            # 业务路由 /api/v1；列表默认 GET+form，复杂筛选可 POST+body
│   ├── scheduler/             # 定时任务；必须复用 Service，禁止重写业务逻辑
│   │   └── jobs.go
│   └── config/                # 配置结构体（cfg.DB / cfg.Redis / cfg.JWT / cfg.Server）
│       └── config.go
├── pkg/                       # 可复用公共组件
│   ├── common/
│   │   ├── errors.go          # AppError 全局错误码及 HTTP 状态码映射
│   │   ├── response.go        # common.Success() / common.Fail() 统一响应
│   │   └── logger.go          # zap 结构化日志 + TraceID，common.Log 全局实例
│   ├── clients/               # 所有第三方 HTTP/SDK 调用必须经此封装，禁止直接调用原始 SDK
│   │   └── external.go        # HTTPClient（带超时、重试、日志）
│   └── database/
│       ├── mysql.go           # GORM 初始化与关闭；是否执行 AutoMigrate 由配置控制
│       └── redis.go           # Redis 初始化与关闭；连接参数与启用开关由配置控制
├── config/
│   └── config.yaml            # 配置源；支持环境变量覆盖；控制 DB/Redis/JWT/Server 及 AutoMigrate/Redis 开关
└── main.go                    # 程序入口

```

**启动顺序**：配置 → 日志 → MySQL → Redis → DI 注入 → 路由注册 → Scheduler → HTTP Server → 信号监听

**关闭顺序**（收到 SIGINT/SIGTERM）：HTTP Server 优雅关闭 → Scheduler 停止 → Redis 关闭 → MySQL 关闭

# Go 后端分层开发规则

## 依赖方向
只允许：

`Controller -> Service -> Repo -> DB/Redis`

允许：
- Service -> Converter
- Service -> Clients

禁止跨层、反向依赖、绕过 Repo 访问业务表。

## 分层职责

### Controller
负责：绑定参数、校验、调用 Service、统一响应、OpenAPI 注释。  
禁止：操作 DB/Repo、写业务逻辑、返回 model.*、处理数据库错误语义。

### Service
负责：业务编排、事务控制、调用 Repo/Converter/Clients、返回 dto.*。  
禁止：依赖 gin.Context、返回 model.*、拼 HTTP 响应、绕过 Repo 访问数据库。

### Repo
负责：MySQL/Redis 访问、返回 model.*、接收 tx。  
禁止：写业务逻辑、处理 HTTP 响应、暴露 DB()、直接 `Updates(req)`。

### Converter
负责：model.* 与 dto.* 转换、脱敏、裁剪。  
禁止：访问数据库、写业务逻辑。

### Model
仅用于数据库映射。允许 `gorm`；禁止 `json/form/binding`；不得直接返回前端。

### DTO
仅用于请求/响应传输。  
请求 DTO 允许：`json/form/binding`  
响应 DTO 允许：`json`  
禁止：`gorm`

## 强制规则
1. Controller / Service / Repo 公开方法首参必须是 `context.Context`
2. Service 返回给 Controller 的结果必须是 `dto.*`
3. 严禁返回 `model.*` 给前端
4. 事务归 Service，Repo 仅接收 `tx`
5. 严禁 Repo 暴露 `DB()`
6. 依赖必须接口注入
7. 更新必须字段白名单，禁止直接 `Updates(req)`
8. Scheduler 必须复用 Service
9. Controller 不处理底层数据库错误，统一走公共错误体系
10. 业务错误统一使用 `AppError`
11. TraceID 必须经 context 全链路透传

## 路由与配置
- 业务前缀：`/api/v1`
- 系统路由：`/healthz` `/ready` `/metrics`
- 列表查询默认 `GET + query/form`
- 复杂筛选才用 `POST + body`
- 配置统一来自 `config/config.yaml`
- 支持环境变量覆盖
- 禁止硬编码配置

## DTO 规则
- 分页请求：`page` `pageSize`
- 分页响应：`list` `total`
- 更新字段需区分未传值/零值时，使用指针
- 响应 DTO 禁止敏感字段

## 错误处理
- Repo：识别/包装底层错误
- Service：补充业务语义
- Controller：只做统一响应

禁止：
- 底层数据库错误直接返回前端
- 绕过全局错误体系
- Controller 手动判断数据库错误并返回 HTTP 状态码

## 禁止事项
- Controller 直接操作 DB/Repo
- Service 依赖 gin.Context
- Service 返回 model.*
- Repo 暴露 DB()
- Repo 写业务逻辑
- Converter 访问数据库
- Model 添加 json 标签
- DTO 添加 gorm 标签
- 直接 `Updates(req)`
- 忽略 error
- 硬编码配置
- 直接调用第三方 SDK，不经统一 clients 封装