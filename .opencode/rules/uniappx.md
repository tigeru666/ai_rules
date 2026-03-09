---
paths:
  - "uniappx_wechat/**/*.{vue,uvue,uts,ts,js}"
  - "uniappx_wechat/**/pages.json"
  - "uniappx_wechat/**/manifest.json"
  - "uniappx_wechat/**/uni.scss"
  - "uniappx_wechat/**/App.vue"
  - "uniappx_wechat/**/main.uts"
  - "uniappx_wechat/**/main.ts"
---



# uni-app x 开发规则

## 定位

uni-app x 用于开发跨端页面与交互，使用 Vue + UTS + uni-app x 组件体系。  
生成代码时，必须优先按 uni-app x 语法、组件、API 和多端约束实现，不能默认按普通 Vue Web 项目处理。

## 原则

- 优先使用 uni-app x 官方支持的语法、组件和 API
- 不要把 uni-app x 当成普通浏览器项目
- 页面、组件、状态、请求、工具函数必须清晰分层
- 涉及跨端能力时，优先考虑多端兼容
- 优先复用项目内已有 easycom 组件

## 目录结构

```text
uniappx_wechat/
├── pages/                 # 页面目录
├── components/            # 可复用组件
├── stores/                # 状态管理
├── composables/           # 复用逻辑
├── api/                   # 接口调用封装
├── utils/                 # 工具函数
├── types/                 # 类型定义
├── static/                # 静态资源
├── pages.json             # 页面路由与窗口配置
├── manifest.json          # 应用配置
└── uni.scss               # 全局样式变量
```

## 页面规则

- 页面文件优先使用 `.uvue`
- 页面只负责展示与交互编排
- 禁止在页面中散写复杂请求逻辑
- 禁止在页面中内联大量类型、工具函数或业务逻辑
- 列表页、详情页、表单页优先拆分

## 组件规则

- 优先使用 uni-app x 官方组件和项目内 easycom 组件
- 通用组件放 `components/`
- 组件保持职责单一
- 组件优先通过 props / emit 通信
- 不要默认使用 Web 专属 DOM 操作方式

## 脚本规则

- 优先使用 UTS / uni-app x 推荐写法
- 不要直接依赖 `window`、`document`
- 平台能力优先使用 uni-app x 官方 API
- 异步逻辑、请求逻辑、数据转换逻辑优先抽到 `api/`、`composables/` 或 `utils/`

## 样式规则

- 优先使用 uni-app x 支持的样式能力
- 不要默认使用只适用于 Web 的 CSS 特性
- 全局变量统一放 `uni.scss`
- 页面样式与组件样式分离

## API 规则

- 所有请求统一收敛到 `api/`
- 禁止页面直接散写请求
- 统一处理：
  - `baseURL`
  - token 注入
  - 错误处理
  - 通用响应适配
- 按模块组织 API，如：
  - `userApi.getList`
  - `userApi.getDetail`
  - `authApi.login`

## 状态规则

- 全局状态统一放 `stores/`
- 页面私有状态不要滥用全局 store
- store 只负责状态与状态变更
- 不要把所有状态堆进一个大 store

## 复用逻辑规则

- 复用逻辑优先提取到 `composables/`
- 不要在多个页面复制相同逻辑
- 表单、分页、筛选、权限判断等逻辑优先抽离

## 配置规则

- 页面路由、导航栏、窗口配置统一放 `pages.json`
- 应用级配置统一放 `manifest.json`
- 全局样式变量统一放 `uni.scss`
- 不要在页面中硬编码可配置项

## 多端规则

- 默认考虑多端运行场景
- 不要只按 H5 行为实现功能
- 涉及平台差异时，应显式隔离实现
- 不要默认引入浏览器专属或单平台依赖

## 禁止事项

- 把 uni-app x 当成普通 Web Vue 项目处理
- 默认使用 `window`、`document`、原生 DOM API
- 页面里直接散写请求
- 页面里堆积大量逻辑
- 组件强耦合具体页面
- 忽略多端兼容
- 忽略项目内已有 easycom 组件
- 随意引入不适合 uni-app x 的 Web 依赖