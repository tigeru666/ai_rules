# 项目协作说明

本项目使用 Claude Code 进行辅助开发。  
请始终遵守 `.claude/rules/` 下的规则文件，优先保证架构一致性、可维护性、可测试性和协作规范一致性。

## 项目概览

- 后端：Go + Gin + GORM + MySQL + Redis
- 前端：Vue 3 + TypeScript + Vite + Pinia + Vue Router + Element Plus + Tailwind CSS
- 桌面端：Wails v2.11.0
- 小程序 / 跨端：uni-app x
- API 描述：OpenAPI

## 总原则

- 遵守分层架构，禁止跨层调用
- 保持目录清晰，避免单文件膨胀
- 优先复用已有模块，不重复造轮子
- 默认补全错误处理，不允许静默失败
- 保持代码简洁、可读、可维护
- 不随意引入新依赖
- 修改代码时尽量保持现有风格一致

## 模块原则

- 后端：`Controller -> Service -> Repo -> DB/Redis`
- 前端：页面、请求、状态、组件分层清晰
- Wails：只负责桌面 UI、系统能力、桥接能力
- uni-app x：优先使用官方语法、组件、API，并考虑多端兼容
- Git：分支、提交、合并流程遵守 `git.md`

## 规则文件

- `.claude/rules/backend.md`
- `.claude/rules/frontend.md`
- `.claude/rules/wails.md`
- `.claude/rules/uniappx.md`
- `.claude/rules/code-style.md`
- `.claude/rules/testing.md`
- `.claude/rules/security.md`
- `.claude/rules/openapi.md`
- `.claude/rules/git.md`

## 使用建议

- 后端开发看 `backend.md`
- 前端开发看 `frontend.md`
- 桌面端开发看 `wails.md`
- uni-app x / 小程序开发看 `uniappx.md`
- 接口描述看 `openapi.md`
- 测试看 `testing.md`
- 安全相关看 `security.md`
- Git 协作看 `git.md`