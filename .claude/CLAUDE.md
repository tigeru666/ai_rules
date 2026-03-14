# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

本项目使用 Claude Code 进行辅助开发。
请始终遵守 `.claude/rules/` 下的规则文件，优先保证架构一致性、可维护性、可测试性和协作规范一致性。

---

## 项目概览

- **后端**：Go 1.23 + Gin v1.10 + GORM v1.25 + MySQL + Redis v9 + Zap + Viper
- **前端**：Vue 3.5 + TypeScript 5.9 + Vite 7 + Pinia + Vue Router + Element Plus 2 + Tailwind CSS 4
- **桌面端**：Wails v2.11.0
- **小程序 / 跨端**：uni-app x
- **API 描述**：OpenAPI（模块化 YAML，位于 `server/docs/openapi/`）

---

## 架构说明

### 后端分层（严格，禁止跨层调用）
```
Controller → Service → Repo → DB / Redis
Service 可调用：Converter / pkg/clients/*
```

**关键约束**（详见 `.claude/rules/backend.md`）：
- `model/` 文件：只允许 `gorm` 标签，**禁止 json/form/binding**
- `dto/request.go`：允许 `json/form/binding`，禁止 `gorm`
- `dto/response.go`：只允许 `json`，禁止 `gorm/binding`
- Controller/Service/Repo 公开方法首参必须是 `context.Context`
- Service 返回 `dto.*`，禁止返回 `model.*`
- 事务逻辑归 Service，Repo 仅接收 `tx *gorm.DB`
- 更新操作必须字段白名单，禁止 `Updates(req)`
- 第三方 SDK 必须经 `pkg/clients/` 封装

### 前端分层（详见 `.claude/rules/frontend.md`）
```
views/（页面） → api/（接口） → utils/request.ts（Axios 封装）
views/（页面） → stores/（Pinia 状态）
```
- 页面禁止直接调用 axios，必须走 `api/` 模块
- 接口返回类型与表单类型分离（`types/` 目录统一管理）
- 常量和枚举统一放 `constants/`
- 组件通过 props/emits 通信，不耦合具体页面

### OpenAPI 文档（详见 `.claude/rules/openapi.md`）
- 一个模块一个 YAML，禁止合并为单文件
- 公共 Schema 在 `common.yaml`，通过 `$ref` 引用
- 涉及接口变更时，必须同步更新对应 YAML

---

## 已实现业务模块

| 模块 | 后端路由前缀 | 前端页面 |
|------|-------------|---------|
| 认证 | `/api/v1/auth` | `views/LoginView.vue` |
| 用户 | `/api/v1/users` | `views/user/` |
| 角色 | `/api/v1/roles` | `views/role/` |
| 菜单 | `/api/v1/menus` | `views/menu/` |
| 配置 | `/api/v1/configs` | `views/config/` |

默认测试账号：`admin1 / Admin@123`

---

## 规则文件索引

| 场景 | 规则文件 |
|------|---------|
| 后端开发 | `.claude/rules/backend.md` |
| 前端开发 | `.claude/rules/frontend.md` |
| 数据库设计 | `.claude/rules/database.md`|
| 接口描述 | `.claude/rules/openapi.md` |
| 安全 | `.claude/rules/security.md` |
| 测试 | `.claude/rules/testing.md` |
| 桌面端 | `.claude/rules/wails.md` |
| uni-app x | `.claude/rules/uniappx.md` |
| Git 协作 | `.claude/rules/git.md` |

---

## 标准执行流程

当用户提出开发需求时，按以下顺序工作：

1. 识别任务类型：后端 / 前端 / OpenAPI / 安全 / 测试 / 数据库设计
2. 读取对应规则文件后再设计
3. 先输出：需求理解、设计方案、影响范围、需新增/修改的文件、是否涉及数据库/接口/前端联调、需询问用户确认的问题
4. 等待用户确认后再进入实现阶段；未确认前禁止输出完整代码、Migration、完整 SQL、OpenAPI 变更内容或可直接执行的实现方案
5. 涉及接口变更 → 同步更新 OpenAPI YAML
6. 涉及表结构变更 → 必须补充 GORM Model 变更，并提供对应 Migration 方案（AutoMigrate、手写 SQL 或独立 migration 文件）
