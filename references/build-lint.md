# 代码规范层施工指南（lint / format / 类型 / EditorConfig / pre-commit）

按 Phase 1 探到的生态选段落执行。**先看 `lang-matrix.md` 确认选型**，再来这里取具体配置。

## 通用纪律

1. **尊重存量**：仓库已有配置就在其上增强，不推翻。
2. **依赖白名单**：只装 `eslint` `prettier` `@biomejs/biome` `ruff`（Python 装法见下）`pre-commit` 及官方插件。装前告知用户。
3. **只做格式无语义修正**：可以跑 `--fix`/`format` 改格式，不重写业务逻辑。
4. **装完验证**：跑一次确认能用，把结果给用户看。

---

## JS / TS

### EditorConfig（通用，先做）
`.editorconfig`：
```ini
root = true
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2
[*.md]
trim_trailing_whitespace = false
```

### 格式化：Prettier
```bash
<pm> add -D prettier   # pm = pnpm/npm/yarn
```
`.prettierrc`（跟随项目习惯，下面是常见默认）：
```json
{ "semi": true, "singleQuote": true, "trailingComma": "all", "printWidth": 100 }
```
加 `package.json` script：`"format": "prettier --write ."`

### Lint：ESLint（TS 项目）
```bash
<pm> add -D eslint @eslint/js typescript-eslint
```
flat config `eslint.config.js`（ESLint 9+）：
```js
import js from '@eslint/js';
import tseslint from 'typescript-eslint';
export default tseslint.config(
  js.configs.recommended,
  ...tseslint.configs.recommended,
);
```
script：`"lint": "eslint ."`
> 若项目已有 `.eslintrc.*`（旧版），沿用旧格式补规则，不要强行迁 flat config，除非用户要。

### 格式化 + Lint 一体：Biome（替代方案，二选一）
```bash
<pm> add -D @biomejs/biome
npx biome init
```
script：`"check": "biome check --write ."`

### 类型检查
确保 `tsconfig.json` 开 `"strict": true`；script：`"typecheck": "tsc --noEmit"`。

### 提交前钩子（husky + lint-staged，与提交纪律层一并做）
见 `build-commit.md`。

---

## Python

### EditorConfig
同上，但 `indent_size = 4`，并加：
```ini
[*.py]
indent_size = 4
```

### 格式化 + Lint 一体：Ruff
安装（优先跟随包管理器）：
```bash
uv add --dev ruff        # uv 项目
# 或 pip install --user ruff
```
在 `pyproject.toml` 追加：
```toml
[tool.ruff]
line-length = 100
target-version = "py312"   # 按实际版本

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B"]  # 基础 + import 排序 + 现代化 + bugbear
```
命令：`ruff check .` / `ruff format .`

### 类型检查
- pyright：`npx pyright`（无需装进项目）或 `uv add --dev pyright`
- 或 mypy：`uv add --dev mypy`，`pyproject.toml` 配 `[tool.mypy]`。
选型跟随项目已有；都没有则推 pyright（快、开箱好）。

### 提交前钩子（Python 生态用 pre-commit 框架）
```bash
uv add --dev pre-commit    # 或 pip install --user pre-commit
```
`.pre-commit-config.yaml`：
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```
装钩子：`pre-commit install`

---

## Go / Rust / 其他

- **Go**：`gofmt`/`goimports` 自带；lint 用 `golangci-lint`（`golangci-lint run`）；钩子用 lefthook 或 pre-commit。
- **Rust**：`cargo fmt` + `cargo clippy` 自带；钩子用 pre-commit。
- **不在表内的生态**：跳过语言特定 lint，只保留 EditorConfig，并在诊断里说明「该生态暂无预置配置，可手动补」。

---

## 自检（施工尾声）

装完后跑一次确认可用，例如：
```bash
<pm> lint         # 或 ruff check . / golangci-lint run
<pm> typecheck    # 或 npx pyright
```
把是否通过、有多少 warning 报给用户。**首次接入 lint 常有大量存量告警属正常**——告诉用户这是存量、可后续渐进清理，不要为了「清零」去大改业务代码。
