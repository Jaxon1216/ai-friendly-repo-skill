---
name: ai-friendly-repo
description: >-
  扫描一个代码仓库的「AI 友好度」，产出可勾选的建设建议清单，在用户授权后按勾选项施工（生成 AGENTS.md / CLAUDE.md /
  ARCHITECTURE.md / GLOSSARY.md、配置 ESLint/Prettier/Ruff 等 linter 与格式化、EditorConfig、
  提交前钩子、Conventional Commits 规范、husky+commitlint+lint-staged、以及项目特异的 git 红线 /
  宿主端环境 / 避坑指南 / 分语言 Code Style）。当用户想让一个项目对 AI Coding 工具（Codex / Cursor /
  Claude Code）更友好、想提升 AI 生成代码的采纳率、想建设 AGENTS.md 或 CLAUDE.md、想给仓库补 lint /
  prettier / husky / commit 规范、问「这个仓库怎么让 AI 更好地理解 / 少改错」「怎么做 AI 友好度建设」「帮我扫一下仓库的
  AI 友好度」时，使用本 skill。核心原则：能被硬脚本（linter / 类型检查 / 钩子）确定性验证的规则优先，减少 AI
  决策幻觉；探测不出的项目语义（宿主端、业务黑话、避坑点）用交互提问 + 占位符让用户补全。
---

# AI 友好度建设 (AI-Friendly Repo)

## 执行定位

按 **扫描 → 建议 → 授权 → 施工** 的四阶段流程建设目标仓库的 AI 友好度。执行时必须先只读扫描，再输出可勾选建议清单，等待用户授权后只施工被勾选的项。

核心执行规则：
- **硬脚本优先**：能被 linter / 格式化 / 类型检查 / git 钩子确定性验证的规则，优先落成配置和脚本。
- **探测优先**：能从仓库文件探测出来的事实自动填；探测不出但用户知道的信息要问用户；用户答不上来的留 `[占位：请补充 xxx]`，不要编造。
- **授权施工**：不要跳过建议清单直接改文件。

```
Phase 1 扫描 (Scan)       → 探测仓库事实，读 references/detection.md
Phase 2 诊断+建议 (Report) → 逐项打分 🟢/🟡/🔴，输出可勾选清单
Phase 3 授权 (Authorize)   → 用户勾选要建设的项；对占位项交互提问
Phase 4 施工 (Build)       → 只对勾选项动手，读 references/build-*.md
```

---

## Phase 1 · 扫描仓库事实

先确认仓库根目录（用户指定的路径，或当前目录）。然后**只读文件、不改任何东西**，探测以下事实。完整探测清单和命令见 `references/detection.md`，核心是：

- **语言与生态**：读 `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml` 等，判定主语言、包管理器（npm/pnpm/yarn/uv/pip…）、是否 monorepo（workspaces / 多 package）。
- **已有的 AI 友好度资产**：是否已有 `AGENTS.md` / `CLAUDE.md` / `README.md` / `ARCHITECTURE.md` / `GLOSSARY.md`；是否已配 ESLint/Prettier/Biome/Ruff、EditorConfig、husky/pre-commit、commitlint。
- **代码规模信号**：有没有超大文件（>400 行）、有没有测试目录。
- **commit 历史风格**：`git log --oneline -30`，看是否已在用 `feat:/fix:` 这类前缀（用来推断「提交规范」的现状）。

探测时要根据实际生态**选对工具**——探到 Python 就查 Ruff/black，探到 JS/TS 才查 ESLint/Prettier，绝不给 Python 仓库推 Prettier。语言→工具链的映射见 `references/lang-matrix.md`。

## Phase 2 · 诊断 + 输出可勾选建议清单

对照下面的**建设清单**逐项打分，用 🟢已具备 / 🟡偏弱可改进 / 🔴缺失 三档，然后输出一张**可勾选的建议清单**给用户。每条建议必须写清三件事：**现状、建议动作、以及本仓库该用哪个具体工具**（基于 Phase 1 的探测结果，不要写通用废话）。

### 建设清单（本 skill 的建设范围）

清单分四组。每组都是「一类相关的建设项」，勾选时用**建设项的名字**回复即可（如「AGENTS.md、Lint、提交钩子」或「全部」）。详细模板和施工规则按需读取对应 reference，不要把所有 reference 一次性读入上下文。

**第一组 · 文档层（全部支持）**
- **AGENTS.md / CLAUDE.md / ARCHITECTURE.md / GLOSSARY.md**
- 生成模板、占位规则、AGENTS.md 精简要求：读 `references/build-docs.md`

**第二组 · 代码规范层（随语言定制，能装依赖的走硬脚本）**
- **格式化 / Lint / 类型检查 / EditorConfig / pre-commit hook**
- 语言到工具链映射：读 `references/lang-matrix.md`
- 配置和安装规则：读 `references/build-lint.md`

**第三组 · 提交纪律层**
- **Conventional Commits 规范 / 提交信息硬钩子**
- commitlint、husky、lint-staged 规则：读 `references/build-commit.md`

**第四组 · 项目特异约定层（无法生成的问用户+占位）**
- **Git 操作红线 / 宿主端运行环境 / 避坑指南 / 分语言 Code Style**
- 这些信息主要来自 Phase 3 问答；落文档时读 `references/build-docs.md`

> **明确不做**：环境护栏层（.env/lockfile/setup 脚本，环境信息改为写进 ARCHITECTURE.md/README）、AI 工具适配层（Skill 同源分发/Rules 迁移/PR 模板/LSP）、一键自验脚本、CI 配置。用户已排除，不要主动扩张。

### 建议清单的输出格式

用一张表 + 勾选框呈现，让用户一眼看懂并直接回复要哪些。示例：

```
## 仓库 AI 友好度诊断（<repo-name>，探测为 pnpm + TypeScript monorepo）

| # | 建设项 | 现状 | 建议动作（本仓库） |
|---|------|------|------|
| 1 | AGENTS.md | 🔴 缺失 | 生成主入口，命令从 package.json scripts 提取，环境/utils 复用约定问你 |
| 2 | Lint | 🟡 有 .eslintrc 但无 TS 规则 | 补 typescript-eslint，接入 pnpm lint |
| 3 | 提交信息硬钩子 | 🔴 无 | 装 husky+commitlint+lint-staged（低风险 dev 依赖） |
| ... | ... | ... | ... |

请回复你想建设的编号（如「1 2 3」或「全部」）。
标 [需你补充] 的项我会在施工时问你几个问题。
```

## Phase 3 · 授权 + 收集占位信息

等用户勾选。**只对勾选的项施工**，没勾的一律不碰。

对勾选项里那些「探测不出、需要用户补充」的信息，在这一步**集中交互式提问**（一次问清，别来回打断）。典型要问的：
- 宿主端：这个项目跑在什么环境/容器？是否跨端（同一套代码跑多端）？用什么调试？
- 大任务后自检：AI 改完代码后，希望它每次必须跑什么命令来自验？
- utils 复用：有没有必须优先复用的公共方法（如统一的网络请求/金额格式化封装），禁止 AI 重复造轮子？
- 避坑：这个项目有没有反复踩的坑、新人常犯的错？
- Git 红线：AI 能不能自己 commit/push？

用户答不上来的，**不要追问逼答**，直接在生成的文件里留 `[占位：请补充 xxx]` 高亮标记，并在施工完成后汇总一份「待你补充清单」。

## Phase 4 · 施工

只对授权的项动手。按类别读对应的施工指南：

- **文档层生成** → 读 `references/build-docs.md`（AGENTS.md / CLAUDE.md / ARCHITECTURE.md / GLOSSARY.md 的模板与占位规则）
- **代码规范配置** → 读 `references/build-lint.md`（各生态的 linter/格式化/类型/EditorConfig/pre-commit 配置）
- **提交纪律** → 读 `references/build-commit.md`（Conventional Commits 规则文案 + husky/commitlint/lint-staged 安装配置）
- **项目特异约定** → 内容主要来自 Phase 3 的问答，按 `references/build-docs.md` 里的对应段落写入 AGENTS.md / ARCHITECTURE.md

### 施工的硬约束（务必遵守）

1. **依赖安装白名单**：只允许安装**低风险 dev 依赖**——`husky`、`lint-staged`、`@commitlint/cli`、`@commitlint/config-conventional`、`eslint`、`prettier`、`@biomejs/biome`、`ruff`、`pre-commit` 及其官方插件。**禁止**安装重型运行时依赖、改动业务依赖版本。安装前先告诉用户要装什么。**装 husky/pre-commit 前先探测 `.husky/` 或 `.pre-commit-config.yaml` 是否已存在——已有钩子只做"增强"，绝不覆盖或删除用户已有的 hook。**
2. **不擅改存量业务代码**：可以生成新配置文件、可以跑 `lint --fix` / `format` 对**格式**做无语义修正，但**不要**重写业务逻辑、不要拆分超大文件（除非用户单独授权）。**跑 `lint --fix` / `format` 前必须确认工作区干净(`git status` 无未提交改动)，且把这类批量格式化改动放进独立 commit，不要和配置文件、文档混在一起——保持 git diff 可读、可回滚。**
3. **占位而非编造**：任何探测不出、用户没答的信息，一律留 `[占位]`，绝不用看似合理的假数据填充。宁可空着让用户补，也不要给出会误导 AI 的错误事实——错误的 AGENTS.md 是负资产。
4. **改完自检**：如果装了 linter/钩子，跑一次验证它们能正常工作（如 `npx husky`、`pnpm lint` 能跑通），把结果告诉用户。
5. **版本以仓库为准**：references 里出现的版本号(如 ESLint 9 flat config、ruff v0.6.0、`py312`)**仅为示例**。实际施工时以仓库现有的 Node/Python/工具链版本为准——先探测 `package.json` / `pyproject.toml` / `.tool-versions` 等，按实测版本选择对应配置写法，切勿照抄示例版本号。
6. **最后交付**：列出「已生成/修改的文件清单」+「待用户补充的占位清单」+「装了哪些依赖」，让用户心里有数。

---

## 参考文件索引

按需读取，不要一次性全读进上下文：

- `references/detection.md` — Phase 1 的完整探测清单与命令
- `references/lang-matrix.md` — 语言 → 工具链映射表（选对 linter/格式化/钩子）
- `references/build-docs.md` — 文档层 + 项目特异约定的模板、占位规则、AGENTS.md 写作要点
- `references/build-lint.md` — 代码规范层各生态的 lint/format/类型/EditorConfig/pre-commit 配置样板
- `references/build-commit.md` — 提交纪律层 Conventional Commits 规则文案 + husky/commitlint/lint-staged 配置
