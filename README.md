# stepwise-review

> 逐步子代理执行 + 独立审查的 agent 工作流 skill —— 主对话只编排，每一步都由
> 全新上下文的独立审查者把关。

适用于 ZCode、Claude Code 等支持子代理派发的编码代理宿主。

## 这是什么

你提出一个需求，这个 skill 把它拆成带**可判定验收标准**的步骤；每一步的实现在一个
新开的**实现子代理**里完成，再由另一个全新的**独立审查子代理**对照验收标准审查
（判定表 + 证据 + 亲自复跑 + 证伪姿态），通过才放行；最后由防锚定的独立**终审**
对照原始需求整体把关。

解决的核心问题：单对话里"自己写自己审"会自我合理化——独立上下文的审查者只看
需求、标准和 diff，才会真挑刺。

## 核心特性

- **计划审查**：独立子代理按五条审你的步骤清单（覆盖完整性 / 顺序依赖 / 原子边界 /
  标准可判定性 / 范围纪律），漏项和排反的顺序在开工前就被拦下；
- **双模式**：完整模式全关卡；单步小任务自动建议**轻量模式**（省计划审查与终审，
  保底"实现者 + 审查者"双关卡），`--light` / `--full` 可强制指定；
- **判定表审查**：逐条验收标准 → 通过 / 不通过 / 未验证 + 证据；验证命令必须由
  审查者**亲自重跑**（不采信实现者输出），并覆盖累积回归、防缓存假绿；
- **三种基线模式全有回滚路径**：git 允许提交（任务域 commit / revert）、git 不允许
  提交（步前快照恢复）、非 git（快照模式）；
- **闭环处置**：两轮打回上限、中途 re-plan（含用户变更需求入口）、终审发现必须
  修复 + 全量回归复验；
- **落盘恢复**：验证命令登记表 / 实现者摘要 / 判定表落盘，会话中断后可接续；
- **非代码任务适配**：文档、调研类任务以产出文件为审查对象，全文交叉比对；
- **受限并行**：仅在独立工作区隔离（git worktree）+ 零交集 + 语义独立时允许交叠
  执行，共享工作区一律串行——审查精度优先于吞吐量；
- **19 个 RED/GREEN 场景自检**内置于 skill 文本，用于变更后的回归验证。

## 安装

```bash
# 用 skills CLI（Vercel labs）；根目录 SKILL.md 可被直接发现
npx skills add yangjinhming10-crypto/stepwise-review
# 例：全局安装到指定 agent、免交互
npx skills add yangjinhming10-crypto/stepwise-review -g -a claude-code -y

# 或手动复制（先建目录再拷贝）
mkdir -p ~/.zcode/skills/stepwise-review && cp SKILL.md ~/.zcode/skills/stepwise-review/   # ZCode
mkdir -p ~/.claude/skills/stepwise-review && cp SKILL.md ~/.claude/skills/stepwise-review/ # Claude Code
```

## 使用

```text
/stepwise-review <需求或任务描述>
/stepwise-review --focus "性能与内存" <需求>
/stepwise-review --commit yes <需求>
/stepwise-review --light | --full <需求>
```

| 参数 | 作用 |
|---|---|
| `--focus "风险面"` | 把关注面注入所有审查子代理（追加项） |
| `--commit yes/no` | 预设基线模式，免掉确认时的提问 |
| `--light` | 强制轻量模式（不满足单步硬条件时忽略并告知） |
| `--full` | 强制完整模式 |

完整的功能走读与使用说明见 [docs/README.md](docs/README.md)，变更记录见
[docs/CHANGELOG.md](docs/CHANGELOG.md)。

## 适用判断

一句话：**错了你会在意的任务才值得用它**。适合多步骤功能开发、带复现命令的 bug
修复、重构、重要改动、要求 diff 干净的场景、可判定验收标准的文档/调研任务；
不适合探索性 spike 和纯问答（没有可审的产出）。纯手动触发——不用即不介入。

## 质量验证

v1.0 经过三个独立评审方（内置子代理 + 两个外部 headless 代理）按统一验收标准
（内部一致性 / 可执行性 / 关卡完整性 / 失败模式覆盖 / 安全性）验收，全部
**通过、零阻断项**。验收方式见 [docs/CHANGELOG.md](docs/CHANGELOG.md) 附记。
