# 语言 → 工具链映射表

Phase 1 探到生态后，用这张表**只推荐该生态的工具组合**。核心纪律：绝不给 Python 仓库推 Prettier/ESLint，也不给 Go/Rust 仓库塞 husky（它们有自己的钩子生态）。当探测到的生态不在表内，走「通用兜底」：只做文档层 + Conventional Commits 规范文案 + EditorConfig，跳过语言特定的 lint。

| 生态 | 探测信号 | 格式化 | Lint | 类型检查 | pre-commit 钩子 | 推荐 commit 工具链 |
|---|---|---|---|---|---|---|
| **JS/TS** | `package.json` | Prettier 或 **Biome**（二选一，若已有其一就沿用） | ESLint(+typescript-eslint) 或 Biome | `tsc --noEmit`（strict） | **husky + lint-staged** | husky + @commitlint/{cli,config-conventional} |
| **Python** | `pyproject.toml` | **Ruff format**（或 black） | **Ruff**（或 flake8） | pyright 或 mypy | **pre-commit 框架**（`.pre-commit-config.yaml`） | pre-commit 里挂 commitizen / commitlint hook |
| **Go** | `go.mod` | gofmt / goimports | golangci-lint | 编译器自带 | lefthook 或 pre-commit | 同左钩子挂 commit-msg 校验 |
| **Rust** | `Cargo.toml` | rustfmt | Clippy | 编译器自带 | pre-commit / cargo-husky | 同左 |
| **其他/混合** | — | 跳过或问用户 | 跳过 | 跳过 | 跳过 | 仅写 Conventional Commits 文案 |

## 选型决策要点

1. **尊重存量**：若仓库已有 Prettier 就别推 Biome，已有 ESLint 就别换；本 skill 的职责是**补齐和规范**，不是推翻用户已有选型。探测到已配置 → 只在 🟡（配置偏弱）时建议增强（如给 ESLint 补 typescript-eslint 规则）。
2. **Biome vs ESLint+Prettier**：新项目/无存量时，若用户想要「一个工具搞定 lint+format 且快」，可推荐 Biome；否则默认 ESLint+Prettier 组合（生态最广）。让用户选，别擅自决定。
3. **monorepo**：命令要按包给（如 `backend/` 用 ruff、`frontend/` 用 biome，参照真实项目常见形态）。lint 配置放根目录还是各包，取决于现有结构，跟随即可。
4. **Python 钩子生态不同**：Python 世界的标准是 `pre-commit` 框架（`.pre-commit-config.yaml`），而不是 husky（husky 属于 Node 生态）。别把 husky 硬塞进纯 Python 仓库——除非该仓库是前后端混合、根目录本就有 `package.json`。

## 「大任务后自检命令」的生态对应（写进 AGENTS.md 的命令表）

AGENTS.md 里那句「每次改完必须跑 xxx 自检」，命令按生态取：

- JS/TS：`pnpm lint && pnpm typecheck`（有测试再加 `&& pnpm test`）
- Python（uv）：`uv run ruff check . && uv run pyright`（有测试再加 `&& uv run pytest`）
- Go：`gofmt -l . && golangci-lint run && go test ./...`
- Rust：`cargo fmt --check && cargo clippy && cargo test`

若用户在 Phase 3 指定了自己的自检命令，以用户的为准。
