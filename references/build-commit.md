# F 层提交纪律施工指南（Conventional Commits + husky/commitlint/lint-staged）

这是「零幻觉」维度的代表：commit 规范落成 **commit-msg 硬钩子** 后，不规范的提交直接被拒，不靠 AI/人的自觉。

## 一 · Conventional Commits 规范文案

写进 AGENTS.md（和/或 CONTRIBUTING.md）。规范结构：

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

**type 取值**（约定式提交 + Angular 惯例）：
| type | 含义 | 版本影响 |
|---|---|---|
| `feat` | 新功能 | MINOR |
| `fix` | 修 bug | PATCH |
| `docs` | 文档 | — |
| `style` | 格式（不影响逻辑，如空格/分号） | — |
| `refactor` | 重构（不改功能不修 bug） | — |
| `perf` | 性能优化 | — |
| `test` | 测试增改 | — |
| `build` | 构建系统/依赖 | — |
| `ci` | CI 配置 | — |
| `chore` | 杂项 | — |
| `revert` | 回滚 | — |

**规则要点**：
- `feat`→MINOR、`fix`→PATCH；带 `BREAKING CHANGE`（footer）或 `type!`（如 `feat!:`）→ MAJOR。
- scope 加在圆括号里给上下文，如 `feat(parser): add array parsing`。
- 只有 `BREAKING CHANGE` 必须大写；type 大小写不敏感但约定小写。

写进 AGENTS.md 的示例段落：
```markdown
## Commit 规范（Conventional Commits）
格式：`<type>(<scope>): <描述>`
type：feat / fix / docs / style / refactor / perf / test / build / ci / chore / revert
破坏性变更用 `type!:` 或 footer `BREAKING CHANGE:`。
示例：`feat(auth): 支持 JWT 登录`、`fix(router): 修复空指针`
[占位：本项目特有的 commit 约定，如「禁止在 commit 尾部带 Co-Authored-By」]
```
> 那条 `[占位]` 很重要——真实项目常有个性化约定（如禁 AI 署名尾注），务必在 Phase 3 问用户。

## 二 · 提交信息硬钩子

### JS/TS：husky + commitlint + lint-staged

安装（全是低风险 dev 依赖）：
```bash
<pm> add -D husky @commitlint/cli @commitlint/config-conventional lint-staged
npx husky init          # 生成 .husky/ 并配好 prepare 脚本
```

`commitlint.config.js`：
```js
export default { extends: ['@commitlint/config-conventional'] };
```

`.husky/commit-msg`（校验 commit 信息）：
```sh
npx --no -- commitlint --edit "$1"
```

`.husky/pre-commit`（暂存区增量 lint）：
```sh
npx lint-staged
```

`package.json` 里配 lint-staged（按生态填命令）：
```json
"lint-staged": {
  "*.{ts,tsx,js,jsx}": ["eslint --fix", "prettier --write"],
  "*.{json,md,css}": ["prettier --write"]
}
```

### Python：pre-commit 框架挂 commit-msg 校验

husky 是 Node 生态工具，纯 Python 仓库改用 `pre-commit` 框架。在 `.pre-commit-config.yaml` 追加 commit 信息校验：
```yaml
  - repo: https://github.com/commitizen-tools/commitizen
    rev: v3.29.0
    hooks:
      - id: commitizen
        stages: [commit-msg]
```
装 commit-msg 钩子：
```bash
pre-commit install --hook-type commit-msg
```
（前端后端混合、根目录有 `package.json` 的项目，也可直接用上面的 husky 方案。）

### Go / Rust
用 lefthook 或 pre-commit 框架挂 commit-msg 校验，规则同 Conventional Commits。

## 施工后验证

- 确认 `.husky/` 下钩子有可执行权限、`prepare` script 已写入 `package.json`。
- 可做一次 dry-run：`echo "bad msg" | npx commitlint` 应报错；`echo "feat: x" | npx commitlint` 应通过。把验证结果告诉用户，证明「硬钩子真的会拦」。
- 提醒用户：钩子只在**本地 git 提交**时生效；如需服务端强制，需在 CI/MR 侧另配（本轮不含 CI）。
