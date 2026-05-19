# teach-as-you-code

`teach-as-you-code` 是一个面向 AI 辅助编程学习的 Skill。它不是让 AI 多说废话，而是在你明确要求时，把代码改动背后的技术选择、底层原理、替代方案和迁移方法讲清楚。

## Why

AI 可以很快把项目做完，但学习者常常只看到结果，不知道它为什么这样改。

这对非计算机专业学生、转专业/自学者、初级开发者尤其明显：代码能跑了，但自己没有参与技术决策，也没有建立“问题 -> 方案 -> 原理 -> 迁移”的认知链路。

`teach-as-you-code` 解决的是这个问题：当你使用 AI 写代码、解释代码或复盘 diff 时，它会把有学习价值的操作转成结构化解释，让你不只是“拿到代码”，还知道这段代码为什么存在、怎么工作、还能怎么写。

## Quick Start

明确说出 `teach-as-you-code` 或 `$teach-as-you-code`：

```text
用 teach-as-you-code 给我解释这个 diff
```

```text
Use $teach-as-you-code to explain this function from a beginner perspective.
```

```text
用 teach-as-you-code 边写边教我，并且从工程师角度解释关键取舍
```

常见用法：

```text
用 teach-as-you-code 解释这段代码
用 teach-as-you-code 看一下最近的 git diff
用 teach-as-you-code 从小白角度讲这次改动
用 teach-as-you-code 逐行讲这个函数
```

## Strict Opt-in

这个 Skill 默认不会自动触发。你必须明确要求使用 `teach-as-you-code`。

| User request | Trigger? |
|---|---:|
| `解释这段代码` | No |
| `我是小白，讲慢一点` | No |
| `Review this diff` | No |
| `用 Teaching Card 格式解释` | No |
| `用 teach-as-you-code 解释这段代码` | Yes |
| `Use $teach-as-you-code to review this diff` | Yes |

这样设计是为了避免普通问题被强行扩展成很长的教学输出。你想学习时再打开它；只想要结果时，它不会打扰你。

## How It Works

Skill 激活后，它会根据你的请求选择一个 Mode、一个 Audience Profile 和一个 Output Depth。

| What you ask | Mode | Default profile | Default depth |
|---|---|---|---|
| `边写边教我` | Teaching Card Mode | Intermediate | Standard |
| `解释这个 diff / 最近 AI 改了什么` | Diff Walkthrough Mode | Intermediate | Standard |
| `解释这段代码 / 这个函数 / 这个文件` | Code Explanation Mode | Intermediate | Standard |
| `我是小白 / 非 CS / 自学` | Mode unchanged | Novice | Standard |
| `从工程师角度 / 看架构风险` | Mode unchanged | Engineer | Standard |
| `简短一点 / summary only` | Mode unchanged | Profile unchanged | Compact |
| `非常详细 / deep dive` | Mode unchanged | Profile unchanged | Deep |
| `逐行讲 / explain everything` | Mode unchanged | Profile unchanged | Exhaustive |

三个 Mode 的重点：

- **Teaching Card Mode**：在有学习价值的原子操作后输出教学卡片。
- **Diff Walkthrough Mode**：解释已有 git diff 或 AI 改动，包括 Reading Path 和 Rebuild Path。
- **Code Explanation Mode**：解释代码片段、函数、类、文件或模块。

三个 Profile 的重点：

- **Novice**：概念优先，适合小白、非 CS、自学者和初级开发者。
- **Intermediate**：推理优先，适合有基础但想补齐设计取舍的人。
- **Engineer**：工程视角，关注边界、风险、测试、维护性和迁移路径。

## When to Use / When Not to Use

适合使用：

- AI 改了代码，但你不知道它具体改了什么。
- 你想复盘一个 git diff，并理解改动顺序和设计原因。
- 你想知道“如果我自己从旧代码改到现在，应该先改哪里，再改哪里”。
- 你在 vibe coding，但不想只得到成品。
- 你想把一次代码改动转化成可迁移的技术知识。

不太适合使用：

- 只是改一个拼写、格式或一行简单代码。
- 你只想要最终答案，不想看解释。
- 时间很紧，只需要快速修 bug。
- 当前上下文很长，而你不想增加额外 token 消耗。

## Example Output

Diff Walkthrough Mode 的输出会像这样：

```md
## Diff Walkthrough Summary: Phase 7.1 边界收紧

Mode: Diff Walkthrough
Profile: Intermediate
Depth: Standard

### 1. Goal
这次改动让字段覆盖率评估更严格，避免隐藏的 LLM fallback 被误算成成功。

### 2. Change Map
- `scripts/run_field_coverage_eval.py`：不再接受“只要执行成功”这个宽松标准，而是要求命中 `single_table_query`。
- `core/single_table.py`：把排序方向前移到 route 对象里。
- `tests/test_phase7_field_coverage.py`：增加“最低值查询应该升序排序”的回归测试。

### 3. Reading Path
先看评估脚本。
Question to answer：这个项目现在把什么行为定义为“真正成功”？

再看 route 对象和 SQL 生成逻辑。
Question to answer：自然语言里的排序意图，是在哪一步变成结构化查询信息的？

### 4. Rebuild Path
1. 先给 route 增加排序方向字段，因为后面的 SQL 生成需要消费一个结构化值。
   Checkpoint：route 对象可以同时表达升序和降序的 top records 查询。

2. 再让 SQL 生成逻辑使用这个 route 字段。
   Checkpoint：“最低 / lowest” 查询会生成 `ORDER BY ... ASC`。

3. 最后收紧评估和测试。
   Checkpoint：如果 fallback 到 LLM，字段覆盖率评估会失败。

### 5. Key Teaching Cards
Teaching Card：防止评估里的静默 fallback
...
```

Teaching Card Mode 的单张卡片通常会覆盖：

- 用了什么技术
- 为什么选它
- 底层原理是什么
- 还能选什么
- 为什么这里最适合
- 怎么迁移到其他场景
- 学完可以做什么小练习

## Installation

Codex 会从 `$CODEX_HOME/skills` 读取本地 Skill。把这个仓库放到对应目录即可。

Windows PowerShell 示例：

```powershell
git clone https://github.com/AvegeFybird/teach-as-you-code.git "$env:CODEX_HOME\skills\teach-as-you-code"
```

macOS / Linux 示例：

```bash
git clone https://github.com/AvegeFybird/teach-as-you-code.git "${CODEX_HOME:-$HOME/.codex}/skills/teach-as-you-code"
```

如果你不是通过 git 安装，也可以让最终目录结构保持为：

```text
<CODEX_HOME>/skills/teach-as-you-code/SKILL.md
```

## Project Structure

```text
teach-as-you-code/
  SKILL.md
  TECHNICAL_SPEC.md
  TECHNICAL_SPEC.zh-CN.md
  agents/
    openai.yaml
  README.md
```

文件说明：

- `SKILL.md`：运行时 Skill 指令，Codex 真正执行时主要读取它。
- `TECHNICAL_SPEC.md`：英文技术规格，面向想实现或迁移该 Skill 的读者。
- `TECHNICAL_SPEC.zh-CN.md`：中文技术规格。
- `agents/openai.yaml`：Codex / OpenAI Agent UI metadata，用于展示名称、简介、默认提示等；核心行为仍由 `SKILL.md` 定义。
- `README.md`：项目入口说明，面向 GitHub 访客。

## Technical Specification

如果你想了解完整设计，包括激活规则、路由规则、Mode/Profile/Depth 的组合、token 预算、大 diff 行为、长会话策略和跨 Agent 迁移方式，请阅读：

- [TECHNICAL_SPEC.md](TECHNICAL_SPEC.md)
- [TECHNICAL_SPEC.zh-CN.md](TECHNICAL_SPEC.zh-CN.md)

这两个文档结构平行，做出来的 Skill 行为应该一致。英文版更适合跨语言实现者，中文版更适合中文用户和中文社区讨论。
