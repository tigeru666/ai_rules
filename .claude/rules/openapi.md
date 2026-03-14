---
paths:
  - "server/internal/controller/**/*.go"
  - "server/docs/openapi/**/*.yaml"
---

# OpenAPI 规则

## 总原则
- 所有公开接口都必须维护 OpenAPI 描述
- OpenAPI 必须与真实实现保持一致
- 接口变更时必须同步更新文档

## 文件规则
- 一个模块一个 YAML，禁止输出整个项目的大一统 `openapi.yaml`
- 推荐路径：`server/docs/openapi/<module>.yaml`
- 文件名与模块名一致，如 `user.yaml`、`role.yaml`、`auth.yaml`
- 公共 schema 单独放 `server/docs/openapi/common.yaml`

## 模块 YAML 要求
- 必须包含：`openapi`、`info`、`tags`、`paths`、`components`
- 需要鉴权的接口必须定义 `BearerAuth`
- 成功响应、失败响应都必须完整定义
- 有鉴权要求的接口必须声明 `security: [{ BearerAuth: [] }]`
- 无需鉴权的接口不要添加 `security`

## Schema 规则
- Schema 使用 PascalCase，建议带模块前缀，如 `UserItem`、`UserCreateReq`
- 请求 schema 与响应 schema 严格分离，禁止混用
- 不得直接暴露数据库 Model 字段
- 分页响应至少包含 `list` 和 `total`

## 公共 Schema
以下结构优先抽到 `common.yaml`，并通过 `$ref` 引用：
- `ApiResponse`
- `ErrorResponse`
- `Pagination`

## Controller 要求
- Controller 负责维护对应模块的 OpenAPI YAML
- 参数来源必须准确标注：`path` / `query` / `body`
- 路由、参数、响应、鉴权要求必须与代码一致

## 接口设计约定
- 路径命名优先使用名词
- 列表查询默认 `GET + query`
- 复杂筛选可使用 `POST + body`
- 创建使用 `POST`
- 更新使用 `PUT`
- 删除使用 `DELETE`

## 输出要求
生成模块 YAML 时，额外说明：
1. 推荐文件名
2. 推荐保存路径
3. 可抽取到 `common.yaml` 的公共 schema

## 禁止事项
- 输出整个项目统一 `openapi.yaml`
- 直接用数据库 Model 作为请求或响应 schema
- 参数来源标错
- 鉴权接口缺少 `security`
- 分页接口缺少 `list` / `total`
- 文档与实现不一致