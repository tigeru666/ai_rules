---
paths:
  - "web/**/*.{ts,vue,js}"
  - "web/**/package.json"
  - "web/**/vite.config.{ts,js}"
  - "web/**/router/**/*.{ts,js}"
  - "web/**/stores/**/*.{ts,js}"
---

## 目录结构

```text
web/
├── src/
│   ├── api/                     # 接口请求层：按模块封装 API，禁止在页面中直接写 axios
│   │   └── user.ts              # 用户相关接口
│   ├── assets/                  # 静态资源（图片、图标、全局样式）
│   │   ├── images/
│   │   └── styles/
│   ├── components/              # 通用组件（跨模块复用）
│   │   ├── common/              # 基础通用组件
│   │   └── business/            # 业务通用组件
│   ├── composables/             # 组合式函数（复用状态与逻辑）
│   │   └── useUser.ts
│   ├── constants/               # 常量定义（状态枚举、路由名、缓存键等）
│   │   └── index.ts
│   ├── layouts/                 # 布局组件（侧边栏布局、顶栏布局等）
│   │   └── default.vue
│   ├── router/                  # 路由定义与守卫
│   │   ├── index.ts
│   │   └── guards.ts
│   ├── stores/                  # Pinia 状态管理
│   │   ├── user.ts
│   │   └── app.ts
│   ├── types/                   # 全局类型声明、接口类型、DTO 类型
│   │   └── user.ts
│   ├── utils/                   # 工具函数（日期、格式化、权限判断、请求封装等）
│   │   ├── request.ts           # Axios 实例与拦截器
│   │   └── index.ts
│   ├── views/                   # 页面级组件，按模块拆分
│   │   └── user/
│   │       ├── list.vue         # 用户列表页
│   │       ├── detail.vue       # 用户详情页
│   │       └── form.vue         # 用户表单页
│   ├── App.vue                  # 根组件
│   └── main.ts                  # 入口文件
├── public/                      # 原样拷贝到构建产物的静态文件
│   └── favicon.ico
├── .env.development             # 开发环境变量
├── .env.production              # 生产环境变量
├── index.html                   # HTML 模板
├── package.json                 # 项目依赖与脚本
├── tsconfig.json                # TypeScript 配置
├── vite.config.ts               # Vite 配置
└── tailwind.config.ts           # Tailwind 配置
```

## 分层约束

### `views/`

负责页面展示与交互编排。

禁止：
- 直接写原始 axios 请求
- 内联复杂业务逻辑
- 内联大量类型定义

### `api/`

负责按模块封装接口调用。

禁止：
- 写页面状态逻辑
- 写页面展示逻辑

### `stores/`

负责全局状态管理。

禁止：
- 承担接口定义职责
- 混入大量页面级逻辑

### `composables/`

负责复用交互逻辑、状态组合、通用页面行为。

### `components/`

负责可复用 UI。

禁止：
- 直接耦合具体页面
- 混入模块私有请求逻辑

### `utils/request.ts`

统一封装 Axios，请求拦截、响应拦截、Token 注入、统一错误处理都在这里完成。

### `types/`

统一维护类型，避免在页面中内联大量类型定义。

## 页面开发规则

- 页面按模块拆分放入 `views/模块名/`
- 列表页、详情页、表单页优先拆成独立文件
- 列表页负责搜索、表格、分页、批量操作编排
- 表单逻辑优先抽到 composable，避免页面文件过重
- 页面中不要直接拼接复杂请求参数转换逻辑，优先抽到 `api/` 或 composable

## 类型规则

- 类型统一维护在 `types/` 或模块类型文件
- 不要在页面中内联大量 `interface` / `type`
- 接口响应类型与表单类型尽量拆分
- 枚举、状态常量、缓存键统一放 `constants/`

## 请求规则

- 禁止在页面中直接写 axios
- 所有请求必须走 `api/` + `utils/request.ts`
- Token 注入、统一错误提示、401 处理在请求层统一完成
- 接口命名按模块组织，如 `userApi.getList`、`userApi.create`

## 组件规则

- 通用组件放 `components/common/`
- 业务复用组件放 `components/business/`
- 单个页面私有组件可放在对应模块页面目录内
- 组件优先通过 props / emits 通信，避免无边界状态共享

## 禁止事项

- 页面直接写 axios
- store 中直接写大量视图逻辑
- component 直接依赖具体页面实现
- 将所有逻辑都堆在一个 `.vue` 文件中
- 内联大量魔法字符串
- 忽略类型定义
- 将接口返回结构直接当表单结构复用而不做区分
