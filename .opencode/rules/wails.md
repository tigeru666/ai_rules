---
paths:
  - "wails_client/**/*.{go,vue,ts,js}"
  - "wails_client/wails.json"
  - "wails_client/index.html"
  - "wails_client/package.json"
  - "wails_client/vite.config.{ts,js}"
---

# Wails v2.11.0 桌面客户端规则

## 定位

Wails 用于桌面客户端，负责：

- 桌面 UI
- 系统能力
- 本地操作
- 本地 API / 远程 API 调用
- 页面与桌面能力之间的桥接

## 原则

- 页面不得散写原始桥接调用
- 本地 API 与远程 API 必须统一封装
- 桌面能力统一由宿主层或桥接层管理
- 页面、桥接层、请求层边界必须清晰
- 禁止混用 Wails v2 与 v3 写法

## 目录结构

```text
wails_client/
├── main.go                     # Wails 启动入口：创建应用实例并启动桌面客户端
├── app.go                      # 宿主对象：保存上下文、管理生命周期、承载少量全局桌面能力
├── wails.json                  # Wails 项目配置：应用名、前端目录、构建参数、平台配置等
├── build/                      # 构建资源：图标、平台打包辅助资源、安装包构建文件
│   ├── appicon.png             # 应用图标
│   ├── darwin/                 # macOS 构建资源
│   └── windows/                # Windows 构建资源
├── bridge/                     # 桌面桥接层：页面调用的本地能力入口，负责窗口、事件、文件、系统能力等桥接
│   ├── app_bridge.go           # 应用级桥接：窗口控制、系统对话框、文件选择、系统能力
│   ├── event_bridge.go         # 事件桥接：前后端事件注册、派发、协调
│   └── window_bridge.go        # 窗口桥接：显示、隐藏、最小化、关闭、置顶等
└── frontend/                   # Wails 桌面端 UI，使用 Vue 构建，由本规则单独约束
    ├── src/
    │   ├── api/                # 请求与桌面调用封装：本地 API、远程 API、桥接调用统一入口
    │   ├── assets/             # 静态资源：图片、图标、样式
    │   ├── components/         # 可复用桌面 UI 组件
    │   ├── composables/        # 复用逻辑：窗口行为、事件监听、表单逻辑、桌面交互
    │   ├── constants/          # 常量：事件名、缓存键、状态值、窗口标识等
    │   ├── layouts/            # 桌面端布局组件
    │   ├── router/             # 路由定义与守卫
    │   ├── stores/             # 状态管理：应用状态、窗口状态、用户状态等
    │   ├── types/              # 类型定义：接口返回类型、桥接返回类型、表单类型等
    │   ├── utils/              # 工具函数：请求适配、桥接调用适配、格式化等
    │   ├── views/              # 桌面页面，按模块拆分
    │   ├── App.vue             # 根组件
    │   └── main.ts             # 前端入口：挂载 Vue 应用、路由、状态管理
    ├── index.html              # 前端 HTML 模板
    ├── package.json            # 前端依赖与脚本
    ├── tsconfig.json           # TypeScript 配置
    └── vite.config.ts          # Vite 配置
```

## 边界

### `main.go`

仅负责启动应用，禁止承载复杂逻辑。

### `app.go`

仅负责宿主上下文与生命周期，禁止堆积大量业务或桥接方法。

### `bridge/`

只负责桌面能力桥接，如窗口、事件、文件、系统对话框。
禁止膨胀成万能层。

### `frontend/`

只负责桌面端 UI。
禁止页面散写桥接调用或请求细节。

## 生命周期

适合放：
- 保存上下文
- 初始化窗口资源
- 注册应用事件
- 清理宿主资源

禁止：
- 堆积过重逻辑
- 重复初始化资源
- 混入页面逻辑

## 桥接规则

- 桥接方法必须清晰、收敛
- 桥接层不负责页面编排
- 桥接方法必须有明确输入输出
- 错误必须可感知，禁止静默失败

## UI 规则

- 页面、组件、状态、类型应清晰分层
- 桌面能力调用统一封装
- 请求调用统一收敛到 `api/`
- 桥接调用统一收敛到 `api/` 或 `utils/`
- 复用逻辑优先放 `composables/`

## API 规则

- 本地 API 与远程 API 必须统一封装
- 禁止页面直接写请求
- 请求拦截、错误处理、鉴权头统一处理
- 按模块组织 API，如：
  - `userApi.getList`
  - `authApi.login`
  - `desktopApi.selectFile`

## 事件规则

- 优先使用事件机制而不是轮询
- 事件名统一语义化，如：`app:ready`、`window:close-request`、`user:updated`
- 监听注册后必须清理

## 配置规则

- 桌面宿主配置以 `wails.json` 为准
- 禁止硬编码桌面环境配置、平台路径、敏感配置

## 禁止事项

- 混用 Wails v2 与 v3
- 页面散写桥接调用
- 页面散写本地 API / 远程 API 请求
- 生命周期方法堆积重逻辑
- 用轮询替代事件
- 忽略跨平台差异
- 让桥接层、页面层、请求层边界失控
