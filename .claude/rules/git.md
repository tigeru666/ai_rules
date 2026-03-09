## 分支策略

| 分支 | 用途 | 说明 |
|------|------|------|
| `main` | 生产环境 | 稳定版本，禁止直接提交 |
| `develop` | 开发环境 | 开发主分支 |
| `feature/*` | 功能开发 | 从develop创建，完成后合并回develop |
| `hotfix/*` | 紧急修复 | 从main创建，修复后合并到main和develop |

## 提交规范

**格式**: `type(scope): description`

### 提交类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(用户): 添加用户登录功能` |
| `fix` | 修复bug | `fix(订单): 修复订单计算错误` |
| `docs` | 文档更新 | `docs(README): 更新安装说明` |
| `style` | 代码格式 | `style(格式): 统一代码缩进` |
| `refactor` | 重构 | `refactor(API): 优化接口结构` |
| `perf` | 性能优化 | `perf(查询): 优化数据库查询` |
| `test` | 测试 | `test(用户): 添加用户模块测试` |
| `chore` | 构建/工具 | `chore(依赖): 更新依赖版本` |

### 提交示例

```bash
# 新功能
git commit -m "feat(认证): 添加JWT认证功能"

# 修复bug
git commit -m "fix(登录): 修复密码验证逻辑错误"

# 文档更新
git commit -m "docs(API): 更新API文档"
```

## 操作规范

- 提交前先拉取最新代码：`git pull`
- 每次提交保持功能单一，避免混合多个改动
- 提交信息使用中文，清晰描述改动内容
- 禁止提交敏感信息（密码、密钥、token等）
- 合并前确保代码通过测试
