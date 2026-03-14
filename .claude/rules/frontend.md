---
paths:
  - "web/**/*.{ts,vue,js}"
  - "web/**/package.json"
  - "web/**/vite.config.{ts,js}"
  - "web/**/router/**/*.{ts,js}"
  - "web/**/stores/**/*.{ts,js}"
---

# 前端规则

## 目录约定
- `views/`：页面组件，负责展示与交互编排
- `api/`：按模块封装接口
- `stores/`：Pinia 全局状态
- `composables/`：复用交互逻辑与状态
- `components/`：通用组件与业务组件
- `types/`：类型定义
- `constants/`：枚举、状态、缓存键
- `utils/request.ts`：统一 Axios 封装

## 分层规则
- 页面只负责展示、交互、调用 API / store / composables
- API 层只负责请求封装，不写页面逻辑
- Store 负责全局状态，不承载大量页面细节
- Composables 负责复用逻辑，不直接耦合具体页面
- Components 保持可复用，优先通过 props / emits 通信

## 请求规则
- 禁止在页面中直接写 axios
- 所有请求必须走 `api/` + `utils/request.ts`
- Token 注入、错误处理、401 处理统一在请求层完成
- 接口按模块命名，如 `userApi.getList`、`userApi.create`

## 页面规则
- 页面按模块放在 `views/模块名/`
- 列表、详情、表单优先拆分独立文件
- 表单逻辑复杂时优先抽到 composables
- 不要在页面内写复杂参数转换和大段业务逻辑

## 类型规则
- 类型统一放 `types/` 或模块类型文件
- 不要在页面中内联大量 `type/interface`
- 接口响应类型与表单类型分离
- 枚举和状态常量统一放 `constants/`

## 强制规则
1. 页面禁止直接写 axios
2. 禁止把所有逻辑堆在一个 `.vue` 文件
3. 禁止将接口返回结构直接当表单结构复用
4. 禁止在 store 中写大量视图逻辑
5. 禁止组件直接耦合具体页面
6. 禁止内联大量魔法字符串
7. 代码风格必须与现有项目保持一致