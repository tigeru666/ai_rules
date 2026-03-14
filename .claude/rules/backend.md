---
paths:
  - "server/**/*.go"
---

# Go 后端规则

## 依赖方向
仅允许：

`Controller -> Service -> Repo -> DB/Redis`

允许：
- Service -> Converter
- Service -> Clients

禁止跨层、反向依赖、绕过 Repo 访问数据库。

## 分层职责
- Controller：参数绑定、校验、调用 Service、统一响应、维护 OpenAPI 注释
- Service：业务逻辑、事务控制、调用 Repo/Converter/Clients
- Repo：MySQL/Redis 数据访问，仅负责持久化
- Converter：model 与 dto 转换、脱敏、裁剪
- Model：仅用于数据库映射，禁止直接返回前端
- DTO：仅用于请求/响应传输，禁止加 gorm 标签

## 强制规则
1. Controller / Service / Repo 公开方法首参必须是 `context.Context`
2. Controller 不直接操作 DB/Repo，不写业务逻辑
3. Service 不依赖 `gin.Context`，不返回 `model.*`
4. Repo 不写业务逻辑，不暴露 `DB()`
5. 事务归 Service，Repo 仅接收 `tx`
6. 严禁返回 `model.*` 给前端
7. 更新必须字段白名单，禁止直接 `Updates(req)`
8. 依赖通过接口注入
9. Scheduler 必须复用 Service
10. 错误统一走 `AppError`
11. TraceID 必须全链路透传

## 目录约定
- `internal/controller`：HTTP 入口
- `internal/service`：业务逻辑
- `internal/repo`：数据访问
- `internal/converter`：对象转换
- `internal/model`：数据库实体
- `internal/dto`：请求/响应对象
- `internal/router`：路由注册
- `internal/middleware`：中间件
- `internal/scheduler`：定时任务
- `pkg/clients`：第三方调用封装
- `pkg/common`：响应、错误、日志
- `pkg/database`：MySQL/Redis 初始化

## 路由与配置
- 业务前缀：`/api/v1`
- 系统路由：`/healthz` `/ready` `/metrics`
- 列表查询默认 `GET + query/form`
- 复杂筛选才用 `POST + body`
- 配置统一来自 `config/config.yaml`，支持环境变量覆盖
- 禁止硬编码配置

## DTO 规则
- 请求 DTO：允许 `json/form/binding`
- 响应 DTO：允许 `json`
- Model 禁止 `json/form/binding`
- 分页请求统一：`page` `pageSize`
- 分页响应统一：`list` `total`
- 更新场景需区分未传值/零值时，优先使用指针
- 响应 DTO 禁止敏感字段

## 禁止事项
- Controller 直接查库
- Service 返回 Model
- Repo 写业务逻辑
- Converter 访问数据库
- Model 加 `json` 标签
- DTO 加 `gorm` 标签
- 忽略 error
- 绕过统一错误体系
- 直接调用第三方 SDK，不经 `clients` 封装