# 项目协作说明

本项目使用 OpenCode 进行辅助开发。
请根据任务类型按需加载规则文件，优先保证架构一致性、可维护性、可测试性和协作规范一致性。

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
- Git：分支、提交、合并流程遵守 `opencode/rules/git.md`

## 规则文件（懒加载）

根据任务类型，按需加载对应规则文件：

| 任务类型 | 规则文件 | 加载方式 |
|----------|----------|----------|
| 后端开发 | `opencode/rules/backend.md` | 自动匹配 |
| 前端开发 | `opencode/rules/frontend.md` | 自动匹配 |
| 桌面端开发 | `opencode/rules/wails.md` | 自动匹配 |
| uni-app x 开发 | `opencode/rules/uniappx.md` | 自动匹配 |
| 接口描述 | `opencode/rules/openapi.md` | 手动引用 |
| 测试 | `opencode/rules/testing.md` | 手动引用 |
| 安全相关 | `opencode/rules/security.md` | 手动引用 |
| Git 协作 | `opencode/rules/git.md` | 手动引用 |

## 使用建议

- 后端开发：参考 `opencode/rules/backend.md`
- 前端开发：参考 `opencode/rules/frontend.md`
- 桌面端开发：参考 `opencode/rules/wails.md`
- uni-app x 开发：参考 `opencode/rules/uniappx.md`
- 接口描述：参考 `opencode/rules/openapi.md`
- 测试：参考 `opencode/rules/testing.md`
- 安全相关：参考 `opencode/rules/security.md`
- Git 协作：参考 `opencode/rules/git.md`

## 懒加载说明

CRITICAL：规则文件使用懒加载方式，仅在相关任务需要时才加载对应文件。

- `opencode.json` 已配置 glob 模式 `opencode/rules/*.md` 自动加载相关规则
- 对于特定规则，可在代码中通过 `@opencode/rules/xxx.md` 引用
- 不要一次性加载所有规则文件，按需使用
