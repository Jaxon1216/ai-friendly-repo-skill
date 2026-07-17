# 文档层 + 项目特异约定 施工指南

本文件给 AGENTS.md / CLAUDE.md / ARCHITECTURE.md / GLOSSARY.md 的模板与写作规则。核心贯穿原则：**探测能填的填、问用户能问的问、都不行就留 `[占位：请补充…]`，绝不编造事实。**

## 占位符规范

统一用这个格式，方便用户事后 grep 补全：
```
[占位：<要填什么的说明>]
```
例如 `[占位：本项目运行的宿主端容器，如「抖音 App 内财经 WebView」]`。施工结束时，把所有 `[占位：...]` 汇总成一份「待补充清单」给用户。

---

## AGENTS.md（AI 主入口）

**这是最高价值的文件。** 写作要点：

- **精炼**，目标 <300 行。它是「高频入口」，不是百科全书——详细内容拆到 ARCHITECTURE.md/GLOSSARY.md 并索引过去。
- **命令表是核心**——AI 90% 的卡点是「不知道用什么命令」。命令从 `package.json` scripts / Makefile 真实提取，不要编。
- 规范尽量写成「可被工具校验」的硬规则，别写「代码要优雅」这种模糊话。
- 用「Do NOT / 必须」而非「建议」——软措辞 AI 会当耳旁风。

推荐结构（按勾选的维度裁剪）：

```markdown
# AGENTS.md — <项目名>

> 一句话：这是什么项目、解决什么问题。[占位或探测填充]

## 0. 研发红线（Git 操作红线，最高优先级）
- [占位：如「除非明确指示，不要直接执行 git commit / push」]
- <其他从 Phase 3 问答得到的红线>

## 1. 技术栈
- 语言 / 框架 / 状态库 / 工具库：<从 package.json 探测>

## 2. 常用命令
| 目的 | 命令 |
|---|---|
| 安装依赖 | <探测：pnpm install / uv sync …> |
| 启动开发 | <探测> |
| 构建 | <探测> |
| 跑测试 | <探测> |
| Lint | <探测> |
| 类型检查 | <探测> |

## 3. 每次大任务后必须自检（关键提示）
- 改完代码必须运行：`<按生态，见 lang-matrix.md>`
- [占位：其他项目特定的验证步骤]

## 4. 项目结构
<从目录探测，标注各目录职责；标注「不要手改」的目录>

## 5. 复用公共方法（禁止重复造轮子）
- [占位：如「网络请求必须走 utils/request，禁直接 import axios」]
- [占位：如「金额格式化走 utils/currency」]

## 6. 代码风格（见下方 build 细则 / 或索引到规范文件）
- <从现有代码归纳 + 占位>

## 7. Commit 规范
遵循 Conventional Commits，见 §Commit 或 CONTRIBUTING。
<若勾选「Conventional Commits 规范」，插入 build-commit.md 的规则摘要>

## 8. 避坑指南 (Traps)
- [占位：本项目反复踩的坑，如「prefetch.ts 禁访问 window/document」]

## 9. 更多文档
- 架构与环境 → ARCHITECTURE.md
- 术语表 → GLOSSARY.md
```

## CLAUDE.md（单一信源，指向 AGENTS.md）

不要与 AGENTS.md 大段重复。只写：
```markdown
# CLAUDE.md

请按顺序读取：
1. `AGENTS.md`（项目全景 + 命令 + 复用约定 + 避坑）
2. `ARCHITECTURE.md`（架构与运行环境）
3. `GLOSSARY.md`（业务术语）

## Hard Rules（违反 = 应被 AI CR 打回）
<把 AGENTS.md §0 红线 + 关键硬规则抽成 5-10 条清单，每条尽量对应一个可校验的 lint/钩子>
```
> 若某些工具（如 Trae）识别 `.trae/rules/project_rules.md` 而非 CLAUDE.md，在交付说明里提醒用户，但本 skill 默认只生成 AGENTS.md + CLAUDE.md。

## ARCHITECTURE.md（架构 + 运行环境）

这里承载用户明确要的「宿主端 / 是否跨端 / 技术栈全景」。结构：

```markdown
# <项目名> 架构

## 1. 运行环境与宿主端
- 运行环境：[占位：如 Node 18 / iOS+Android WebView / 浏览器]
- 宿主端：[占位：如「抖音 App 内财经容器」/「独立 Web 站点」]
- 是否跨端：[占位：是/否；若是，说明同一套代码跑哪些端]
- 调试方式：[占位：如某某 Console / Chrome DevTools]

## 2. 技术栈全景
<从 package.json/pyproject 探测：语言、框架、状态管理、构建工具、关键依赖>

## 3. 数据流 / 核心链路
[占位：关键链路，如「A 组件 → B store → C 接口」。可用飞书画板/mermaid，交由用户确认]

## 4. 关键决策点
| 决策点 | 文件 | 说明 |
|---|---|---|
| [占位] | [占位] | [占位] |

## 5. 错误码 / API 契约
[占位或探测 proto/openapi 位置]
```

## GLOSSARY.md（术语表，几乎全占位）

AI 探测不出业务黑话。生成一个带分类骨架的空表，靠 Phase 3 问答 + 占位填充：

```markdown
# <项目名> 术语表

## 框架 / 平台
| 术语 | 含义 | 代码位置 |
|---|---|---|
| [占位] | [占位] | [占位] |

## 业务
| 术语 | 含义 |
|---|---|
| [占位] | [占位] |

## 端 / 容器 / 数据
| 术语 | 含义 |
|---|---|
| [占位] | [占位] |
```

若 Phase 1 扫到高频出现但无解释的专有名词/缩写，可以把词条名先列进去，含义留占位——这样用户补起来最省力。

---

## 项目特异约定的内容来自哪里

- **Git 操作红线** → Phase 3 问用户「AI 能否自己 commit/push」，写入 AGENTS.md §0。
- **宿主端 / 运行环境** → Phase 3 问用户，写入 ARCHITECTURE.md §1。
- **避坑指南** → Phase 3 问用户「有没有反复踩的坑」，写入 AGENTS.md §8；答不出留占位。
- **分语言 Code Style** → 部分可从现有代码归纳（命名习惯、注释语言、docstring 风格），写入 AGENTS.md §6 或独立索引；细则可参照 build-lint.md 里 linter 已强制的部分，文档只写 linter 管不到的约定（如「注释解释 WHY 而非 WHAT」「docstring 用 Google 风格」）。

## 反模式（务必避免）

- ❌ AGENTS.md 写成营销文案「本项目致力于…」——AI 要的是约束和事实。
- ❌ AGENTS.md 与 CLAUDE.md 内容重复——让 CLAUDE.md 指向 AGENTS.md。
- ❌ Hard Rules 写成「建议」——软措辞 AI 不当真。
- ❌ 用假数据填占位——错误的文档是负资产，会误导 AI。
- ❌ 一次性生成一堆文档却全是占位——优先把探测得到的真实信息填满，占位只留给确实探测不出的。
