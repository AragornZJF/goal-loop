# goal-loop

> 从**澄清需求**、**规划**到**自动化构建**的 Claude Code skill 流水线 CLI —— 想清楚 → 写下来 → 干出来。

`goal-loop` 把三套方法论融合为一条流水线，让 AI 替你完成「想清楚 → 写下来 → 干出来」：

| 层 | 方法论 | 做什么 | 产出 |
|---|---|---|---|
| **澄清需求** | brainstorming（头脑风暴） | 苏格拉底式反向提问，把模糊想法拆成最小关注单元 | `specs/<topic>.md` |
| **规划** | writing-plans（计划编写） | 对比 specs ↔ 现有代码，列出细粒度任务清单 | `IMPLEMENTATION_PLAN.md` |
| **执行** | Ralph Loop | 每轮清空上下文，选任务 → 实现 → 测试 → 提交 → 推送 | 真实代码 + git commit |

**一句话**：你说"我想做个 X"，goal-loop 会先把 X 问清楚、再拆成可执行计划、最后挂一条无人值守循环把 X 跑成代码。

---

## 安装

```bash
npm install -g goal-loop
```

> 需要 Node.js ≥ 18。

---

## 快速上手

### 1. 在你的项目里初始化

```bash
cd your-project
goal-loop init .
```

这会在当前目录生成：

```
your-project/
├── .claude/skills/goal-loop/    ◀ skill 全部资源（SKILL.md / phases / prompts / scripts / templates）
├── loop.sh                      ◀ wrapper，转发到 skill 内 loop.sh
└── clean-state.sh               ◀ wrapper，转发到 skill 内 clean-state.sh
```

### 2. 用触发词激活（推荐）

在 Claude Code 会话里说：

> "帮我**探索需求**：我想做一个 XXXXX。"

skill 会根据项目文件状态自动路由：

| 当前文件状态 | 进入阶段 |
|---|---|
| 没有 `specs/` | Phase 1（澄清需求） |
| 有 `specs/`，无 `IMPLEMENTATION_PLAN.md` | Phase 1→2 过渡 |
| 有 `IMPLEMENTATION_PLAN.md` | Phase 2→3 过渡（提示启动构建循环） |

| 阶段 | 触发词 |
|------|--------|
| Phase 1（需求/设计探索） | 探索需求、探索设计、用户意图、先探索再实现、需求澄清 |
| Phase 2（实现计划编制） | 写实现计划、写计划、做计划、实现计划 |
| Phase 3（无头构建循环） | goal loop、自主构建、Ralph 循环、构建循环、自动构建 |

### 3. 或直接启动循环

```bash
goal-loop build 20     # Phase 3：无头构建，最多 20 轮
goal-loop plan 5       # Phase 2：无头规划，最多 5 轮
```

首次运行 `loop.sh` 会自动创建 `AGENTS.md` / `build/` / `specs/`，并尝试从 `build/package.json` 检测 build/test/lint 命令填入 `AGENTS.md`。

---

## 命令参考

| 命令 | 说明 |
|---|---|
| `goal-loop init [dir] [--force]` | 在目标目录（默认当前目录）生成 skill + wrapper。`--force` 覆盖已存在文件 |
| `goal-loop build [N]` | 透传到 `./loop.sh build`，无头构建循环（`N` = 最大轮次） |
| `goal-loop plan [N]` | 透传到 `./loop.sh plan`，无头规划循环（仅迭代计划，不实现代码） |
| `goal-loop clean [--confirm]` | 透传到 `./clean-state.sh`，清空项目状态 |
| `goal-loop --help` / `-h` | 帮助 |
| `goal-loop --version` / `-v` | 版本 |

---

## 数据流（文件即状态）

```
 用户对话（Phase 1）
        │
        ▼
   specs/*.md ──┐
                │ (Phase 2: 交互式 或 ./loop.sh plan)
                ▼
   IMPLEMENTATION_PLAN.md ──┐
                            │ (Phase 3: ./loop.sh build)
                            ▼
                    build/ 内代码 + git history
                            │
                            └── AGENTS.md（测试命令做反压闸门）
```

每一阶段的产物就是下一阶段唯一的输入；**文件就是循环的全部记忆，进程本身无状态**。

---

## 平台要求

- **Claude Code CLI**：`loop.sh` 内部调用 `claude -p`，需已安装 [Claude Code](https://claude.com/claude-code)。
- **Bash 环境**：`loop.sh` / `clean-state.sh` 为 bash 脚本（用了 `sed -i`、`git`、`du` 等），需 **Git Bash / WSL / macOS / Linux**。Windows 原生 cmd / PowerShell 无法直接运行。
- **Git**：构建循环每轮会 `git commit` + `git push`。

---

## 安全（运行循环前必读）

`./loop.sh` 以 `claude --dangerously-skip-permissions` 运行，**绕过 Claude 的所有权限提示**：

- **沙箱是唯一安全边界**；沙箱外运行会暴露凭据、浏览器 cookie、SSH 密钥和访问令牌。
- 每轮自动 `git push` 到当前分支，**请确保是隔离分支，不要在生产分支上运行**。
- 推荐在 Docker / E2B / Fly 等隔离环境中使用，并配置最小权限 API 密钥。

**逃生口**：

- `Ctrl+C` 停止循环
- `git reset --hard` 回滚未提交改动
- 删除 `IMPLEMENTATION_PLAN.md` 重新跑 Phase 2（计划按设计是可丢弃的）

---

## License

MIT
