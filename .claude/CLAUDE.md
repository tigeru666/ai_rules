# CLAUDE.md

请始终遵守 `.claude/rules/` 下的规则文件，优先保证架构一致性、可维护性、可测试性和协作规范一致性。

## 项目概览
- 后端：Go 1.23 + Gin + GORM + MySQL + Redis + Zap + Viper
- 前端：Vue 3 + TypeScript + Vite + Pinia + Vue Router + Element Plus + Tailwind CSS
- 桌面端：Wails
- 跨端：uni-app x
- API 文档：OpenAPI（`server/docs/openapi/`）

## 架构约束

### 后端
`Controller → Service → Repo → DB/Redis`

- 禁止跨层调用
- `model` 只允许 `gorm` 标签，禁止 `json/form/binding`
- 禁止使用 GORM 级联和外键约束，关系仅通过显式字段维护
- 数据表必须有表注释，字段必须有字段注释
- `dto/request` 允许 `json/form/binding`，禁止 `gorm`
- `dto/response` 只允许 `json`，禁止 `gorm/binding`
- Controller / Service / Repo 公开方法首参必须是 `context.Context`
- Service 返回 `dto.*`，禁止返回 `model.*`
- 事务归 Service，Repo 仅接收 `tx *gorm.DB`
- 更新必须字段白名单，禁止 `Updates(req)`
- 第三方 SDK 必须经 `pkg/clients/` 封装

### 前端
`views → api → utils/request`
`views → stores`

- 页面禁止直接调用 axios，必须走 `api/`
- 接口类型与表单类型分离，统一放 `types/`
- 常量、枚举统一放 `constants/`
- 组件通过 props / emits 通信，禁止耦合页面

### OpenAPI
- 按业务域或相似功能组织 YAML，不强制一个模块一个文件
- 优先复用已有 YAML，没有再新建
- 公共 Schema 放 `common.yaml`，统一通过 `$ref` 引用
- 接口变更必须同步更新 OpenAPI

## 已实现模块
- 认证：`/api/v1/auth`
- 用户：`/api/v1/users`
- 角色：`/api/v1/roles`
- 菜单：`/api/v1/menus`
- 配置：`/api/v1/configs`
- 统计：`/api/v1/statistics`
- 系统指标：`/api/v1/system`

默认测试账号：`admin1 / 123456`

## 规则文件
- 后端：`.claude/rules/backend.md`
- 前端：`.claude/rules/frontend.md`
- 数据库：`.claude/rules/database.md`
- OpenAPI：`.claude/rules/openapi.md`
- 安全：`.claude/rules/security.md`
- 测试：`.claude/rules/testing.md`
- Wails：`.claude/rules/wails.md`
- uni-app x：`.claude/rules/uniappx.md`
- Git：`.claude/rules/git.md`

## 执行流程
1. 先识别任务类型：后端 / 前端 / OpenAPI / 安全 / 测试 / 数据库设计
2. 先读取对应规则文件，再进行设计
3. 先输出：需求理解、设计方案、影响范围、需新增/修改的文件、是否涉及数据库/接口/前端联调、需确认的问题
4. 未确认前，禁止输出完整代码、Migration、完整 SQL、OpenAPI 变更内容或可直接执行的实现方案
5. 输出代码或实现方案后，不得直接结束；必须再次自检，检查是否存在 bug、遗漏改动、规则冲突、分层违规、字段误用、类型问题、边界条件问题及可测试性问题
6. 自检通过后，如涉及接口变更，必须同步更新 OpenAPI
7. 涉及表结构变更时，必须同步更新 GORM Model，并提供 Migration 方案
8. 8. 全部完成并确认无误后，按 `.claude/rules/git.md` 规范在正确分支提交 Git；禁止直接在 `main` 提交，提交前需确认当前分支并同步最新代码，提交信息必须清晰说明变更范围与目的