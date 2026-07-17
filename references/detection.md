# Phase 1 · 仓库事实探测清单

目标：**只读不写**，用最少的命令把仓库的客观事实摸清楚，作为 Phase 2 打分和 Phase 4 选型的依据。

## 1. 语言与生态（决定后续所有工具选型）

按存在的清单文件判定主语言与包管理器：

| 探测文件 | 生态 | 进一步看 |
|---|---|---|
| `package.json` | JS / TS | 有无 `"workspaces"`、`packageManager` 字段、`pnpm-workspace.yaml` → monorepo；有无 `tsconfig.json` → TS |
| `pyproject.toml` | Python | 有无 `uv.lock`（uv）/ `poetry.lock`（poetry）/ `requirements.txt`（pip） |
| `go.mod` | Go | `go.work` → 多模块 |
| `Cargo.toml` | Rust | `[workspace]` → monorepo |
| `pom.xml` / `build.gradle` | Java/Kotlin | maven / gradle |

命令参考：
```bash
ls -a <repo>                              # 根目录全貌
cat <repo>/package.json 2>/dev/null       # 看 scripts / deps / workspaces / packageManager
cat <repo>/pyproject.toml 2>/dev/null
find <repo> -maxdepth 2 -name "tsconfig*.json" -o -name "pnpm-workspace.yaml" 2>/dev/null
```

**从 `package.json` 的 `scripts` 里提取真实命令**——这是 AGENTS.md「命令表」的黄金来源，比任何模板都准。dev/build/test/lint/typecheck 各对应哪条命令，照抄。

## 2. 已有的 AI 友好度资产（决定打分 🟢/🟡/🔴）

```bash
# 文档类
ls <repo>/AGENTS.md <repo>/CLAUDE.md <repo>/README.md <repo>/ARCHITECTURE.md <repo>/GLOSSARY.md 2>/dev/null
wc -l <repo>/AGENTS.md 2>/dev/null          # >300 行 → 偏胖，打 🟡

# 规范类（存在即说明已配）
ls <repo>/.eslintrc* <repo>/eslint.config.* <repo>/.prettierrc* <repo>/biome.json 2>/dev/null
ls <repo>/.editorconfig <repo>/ruff.toml 2>/dev/null
grep -l "\[tool.ruff\]" <repo>/pyproject.toml 2>/dev/null

# 钩子类
ls -d <repo>/.husky 2>/dev/null
ls <repo>/.pre-commit-config.yaml <repo>/commitlint.config.* <repo>/.commitlintrc* 2>/dev/null
grep -o '"lint-staged"' <repo>/package.json 2>/dev/null
```

打分口径：
- 文件缺失 → 🔴
- 文件在但内容空泛 / 是模板默认值 / 与代码不一致 / AGENTS.md >300 行 → 🟡
- 文件在、内容具项目特异性、与代码一致 → 🟢

## 3. 代码规模与测试信号

```bash
# 超大文件（AI 单次上下文吃不下，>400 行提示，>600 行强烈建议拆）
find <repo>/src -name "*.ts" -o -name "*.tsx" -o -name "*.py" 2>/dev/null | xargs wc -l 2>/dev/null | sort -rn | head -10

# 测试目录
ls -d <repo>/test <repo>/tests <repo>/__tests__ 2>/dev/null
```

> 注意：超大文件治理（D3）本轮不在建设范围内，但可以在诊断里作为「观察项」提一句，不强制施工。

## 4. Commit 历史风格（提交规范现状）

```bash
git -C <repo> log --oneline -30 2>/dev/null
```

看前缀：如果多数已是 `feat:/fix:/docs:` → 提交规范打 🟢或🟡（可能不统一）；如果是自由文本 → 🔴，建议引入 Conventional Commits。也留意有没有 `Co-Authored-By` 之类用户可能想禁掉的尾注。

## 5. 跨端 / 宿主端信号（运行环境，多半探测不出，需问用户）

可以扫一些线索但不要下结论：`package.json` 依赖里有没有多端框架（React Native / Taro / uni-app / 小程序）、有没有 `*.config.ts` 里出现多 target。**最终以问用户为准**——宿主端是什么容器、是否同一套代码跑多端，这类语义 AI 猜不准。
