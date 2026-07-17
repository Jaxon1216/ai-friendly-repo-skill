# AI-Friendly Repo Skill

## 中文版

`ai-friendly-repo` 是一个面向 AI Coding 工具的仓库建设 Skill，用来扫描并提升代码仓库的「AI 友好度」。它适合配合 Codex、Cursor、Claude Code 等工具使用，让 AI 更准确地理解仓库、遵守规范、完成修改，并在提交前自我验证。

它的工作方式是：先扫描目标仓库，输出缺失项和改进建议，再给出可勾选清单，最后只在用户明确授权后施工。目标不是“讨好 AI”，而是把隐性的工程约定变成 AI 和新人都能遵守的清晰规则。

### 它能做什么

这个 Skill 会补齐 AI Coding 工具最需要的仓库信息和硬约束：

- `AGENTS.md`、`CLAUDE.md` 等 AI 入口文档
- `ARCHITECTURE.md`、`GLOSSARY.md` 等架构和术语文档
- 基于真实技术栈的 formatter、linter、typecheck、EditorConfig 配置
- `husky`、`lint-staged`、`commitlint` 等提交前和提交信息校验
- 项目特异的运行环境、避坑指南、复用约定、Git 操作红线

核心信念是：

> AI 读不懂的地方，新人通常也读不懂；AI 容易改错的地方，人类也容易踩坑。

### 什么时候使用

当你希望一个仓库更适合 Codex、Cursor、Claude Code 或其他 AI Coding 工具协作时，可以使用这个 Skill。

适合的场景包括：

- 想让 AI 更快理解仓库结构和开发命令
- 想生成或改进 `AGENTS.md` / `CLAUDE.md`
- 想补齐 lint、format、typecheck、commit hook
- 想把团队里的隐性约定沉淀成明确文档
- 想减少 AI 因命令缺失、架构不清、避坑未写而产生的错误修改

典型触发语：

- “帮我扫一下这个仓库的 AI 友好度”
- “这个仓库怎么让 AI 更好地理解”
- “帮我补 AGENTS.md / CLAUDE.md”
- “帮这个项目建设 lint / prettier / husky / commit 规范”

### 安装

把这个仓库 clone 到你的 AI Coding 工具支持的 skills 或 rules 目录：

```bash
git clone https://github.com/Jaxon1216/ai-friendly-repo-skill.git ai-friendly-repo
```

如果你的工具支持 Skill 目录，可以把仓库放到对应目录下；如果你的工具使用 rules 文件，也可以参考 `SKILL.md` 和 `references/` 里的流程，把它迁移成对应工具的规则格式。

Skill 入口文件是：

```text
SKILL.md
```

### 使用方式

进入你想建设的目标仓库，然后让 AI Coding 工具扫描 AI 友好度：

```text
帮我扫一下这个仓库的 AI 友好度
```

这个 Skill 遵循四阶段流程：

| 阶段 | 名称 | 说明 |
|---|---|---|
| 1 | 扫描 | 只读探测语言、包管理器、scripts、文档、lint 工具、hooks、测试和 commit 历史 |
| 2 | 诊断 | 对各建设项打分，标出已具备、偏弱或缺失，并输出可勾选清单 |
| 3 | 授权 | 等用户选择要建设的项，并收集无法自动探测的项目信息 |
| 4 | 施工 | 只修改授权项，并总结生成文件、依赖和待补充占位 |

它不会静默改项目。它会先告诉你发现了什么，再问你要建设什么。

### 建设范围

#### 文档层

- `AGENTS.md`：AI 主入口，包含命令、结构、规则、复用约定和避坑指南
- `CLAUDE.md`：Claude Code 特定读取顺序和硬规则，不与 `AGENTS.md` 大段重复
- `ARCHITECTURE.md`：运行环境、宿主容器、数据流、关键决策点和 API 契约索引
- `GLOSSARY.md`：业务、平台、框架和端容器术语表

#### 代码规范层

Skill 会根据探测到的生态选择工具：

- JavaScript / TypeScript：Prettier、Biome、ESLint、`tsc`
- Python：Ruff、black-compatible formatting、pyright、mypy
- Go：gofmt、golangci-lint
- Rust：rustfmt、Clippy
- Java / Kotlin：项目已有的 Maven 或 Gradle 检查

同时也支持 `.editorconfig` 和合适的 pre-commit 配置。

#### 提交纪律层

- Conventional Commits 规范
- `commitlint` 配置
- `husky` `commit-msg` hook
- `lint-staged` 暂存区检查

它可以把提交信息规范和暂存区检查做成硬约束，减少“文档写了但没人遵守”的情况。

#### 项目特异约定

有些信息不能从文件里安全推断，Skill 会询问用户，或者留下明确占位：

- Git 操作红线
- 宿主端或运行容器
- 跨端约束
- 必须复用的公共方法
- 已知避坑点和禁用模式
- lint 无法覆盖的团队代码风格

占位格式统一为：

```text
[占位：请补充 xxx]
```

### 安全原则

这个 Skill 故意设计得比较保守：

- 先扫描，再修改。
- 先授权，再施工。
- 只编辑用户勾选的建设项。
- 探测不到的信息不编造，只询问或留占位。
- 优先使用可验证的脚本规则，而不是模糊文档。
- 不盲目覆盖已有 hooks。
- 不改业务逻辑。
- 大规模格式化改动和配置/文档改动分开处理。

一句话：硬脚本优先，文档其次，绝不靠猜。

### 仓库结构

```text
.
├── SKILL.md
└── references
    ├── build-commit.md
    ├── build-docs.md
    ├── build-lint.md
    ├── detection.md
    └── lang-matrix.md
```

`SKILL.md` 定义主流程和决策规则。

`references/detection.md` 定义扫描阶段的只读探测清单。

`references/lang-matrix.md` 维护语言和工具链映射。

`references/build-docs.md` 提供文档模板和占位规则。

`references/build-lint.md` 提供 formatter、linter、typecheck 和 hook 配置指南。

`references/build-commit.md` 提供 Conventional Commits 和提交钩子指南。

### 设计原则

这个仓库把“给人看的项目说明”和“给 AI 执行的操作指令”分开维护：`README.md` 负责解释项目定位、使用方式和设计理念，`SKILL.md` 负责描述触发后必须执行的流程和约束。这样可以减少 Skill 被加载后的上下文占用，也让执行指令更稳定。

#### 硬脚本优先

如果一个规则能被 formatter、linter、typechecker 或 git hook 校验，就优先落成脚本，而不是只写进文档。脚本确定，文档容易被忽略或误读。

#### 探测优先

能从仓库探测出来的就自动填；只有用户知道的就提问；暂时拿不到的就留下清晰占位。

#### AI 友好就是新人友好

让 AI 写错代码的上下文缺失，往往也会拖慢新人上手。这个 Skill 把 AI 友好度当成一种实际的代码库质量指标。

### 维护方式

扩展这个 Skill 时，保持 `SKILL.md` 精简，把详细施工指南放进 `references/`。

推荐维护方式：

- 新增扫描逻辑，改 `references/detection.md`
- 新增语言或工具映射，改 `references/lang-matrix.md`
- 新增文档模板，改 `references/build-docs.md`
- 新增 formatter、linter、hook 配置，改 `references/build-lint.md`
- 新增提交规范，改 `references/build-commit.md`

默认不要扩大范围。当前 Skill 明确不主动覆盖 CI 配置、运行时依赖升级、环境初始化脚本、PR 模板、AI 工具规则迁移等内容，除非用户明确要求。

