# teach-as-you-code 技术规格书

Version 1.0 / 2026-05-18

## 1. 文档目的

本文是 `teach-as-you-code` Skill 的技术规格书。

它面向两类读者：

- 想根据这份文档实现这个 Skill 的实现者
- 想借鉴这个设计模式来做自己 Skill 的参考者

这不是单纯的产品想法文档，而是一份面向 Skill 实现的 prompt 与行为规格书。它定义了这个 Skill 何时介入、如何路由、如何输出、如何控制长度，以及如何在不同用户水平下调整解释方式。

## 2. 设计概览

`teach-as-you-code` 的目标是帮助用户从 AI 辅助编程中学习，而不是让每一个普通编程请求都变成很长的教学输出。

它有三个核心设计：

1. 严格显式激活
2. 双轴解释模型：`Mode x Audience Profile`
3. 面向大 diff、长代码、长会话的输出安全阀

### 2.1 总体流程

```text
用户请求
  -> 是否明确要求使用 teach-as-you-code 或“这个 Skill”？
      -> 否：正常回答，不使用本 Skill。
      -> 是：进入 Mode 和 Audience Profile 路由。
  -> 判断 Mode：
      -> 未来编码过程：Teaching Card Mode
      -> 已有改动 / diff：Diff Walkthrough Mode
      -> 代码片段 / 文件 / 模块：Code Explanation Mode
  -> 判断 Audience Profile：
      -> 小白 / 非 CS / 要求详细：Novice
      -> 未指定 / 有一定基础：Intermediate
      -> 工程视角 / review / 风险 / 取舍：Engineer
  -> 判断输入规模：
      -> 小：一张聚焦卡片
      -> 中：按学习单元分组
      -> 大：总览 + 精选卡片
  -> 输出结构化解释
  -> 接受上下文内控制，如“简短一点”“非常详细”“逐行讲”“换成工程师档”“关闭”
```

### 2.2 双轴模型

```text
Mode = 解释什么场景。
Audience Profile = 以什么深度和视角解释。
```

Modes：

- Teaching Card Mode：在 AI 编码过程中解释关键步骤。
- Diff Walkthrough Mode：解释已有改动、diff 或最近的 AI 修改。
- Code Explanation Mode：解释用户提供的代码、文件、函数、类或模块。

Audience Profiles：

- Novice：小白档，概念优先，非常详细。
- Intermediate：有基础档，推理优先，默认档。
- Engineer：工程师档，取舍优先，关注架构、风险和验证。

## 3. 激活策略

本 Skill 严格 opt-in。

只有当用户明确说 `teach-as-you-code`，或者在当前上下文中明确说“这个 Skill”且“这个 Skill”指的就是 `teach-as-you-code` 时，才使用本 Skill。

不要因为普通编码请求、普通代码解释、普通 diff review、普通 debug 问题，或者单独要求 Teaching Card/Profile 风格解释而自动触发本 Skill。

### 3.1 激活矩阵

| 用户请求 | 是否激活 Skill | 原因 |
|---|---:|---|
| “讲一下这段代码” | 否 | 普通代码解释。 |
| “我是小白，讲一下这段代码” | 否 | 只有人群信号，没有明确使用 Skill。 |
| “从工程师角度讲讲” | 否 | 只有人群信号，没有明确使用 Skill。 |
| “帮我看看这个 diff” | 否 | 普通 diff review。 |
| “边写边教我” | 否 | 教学请求，但没有明确使用 Skill。 |
| “开启 Teaching Card Mode” | 否 | 只是命名格式，不是明确使用 Skill。 |
| “用 Teaching Card 格式解释” | 否 | 只是格式请求，不是明确使用 Skill。 |
| “用 teach-as-you-code 讲一下这段代码” | 是 | 明确使用 Skill。 |
| “用这个 Skill 讲一下这段代码” | 是 | 当前上下文明确指向本 Skill 时成立。 |
| “用 teach-as-you-code 的 Diff Walkthrough Mode 复盘” | 是 | 明确使用 Skill + Mode。 |
| “用 teach-as-you-code 的小白档讲” | 是 | 明确使用 Skill + Profile。 |
| “用 teach-as-you-code 的工程师档看这个 diff” | 是 | 明确使用 Skill + Profile + Mode。 |

### 3.2 激活范围

默认只对当前请求生效。

只有当用户明确扩大范围时，才跨步骤持续：

- 当前功能 / 当前任务
- 当前会话
- 当前项目，如果平台支持持久项目指令

用户说以下内容时停止使用：

- “关闭 teach-as-you-code”
- “停止使用这个 Skill”
- “只给我结果”
- “暂时不要 Teaching Cards”

## 4. 激活后的路由规则

当 Skill 已经被明确激活后，按下表路由。

| 激活后的信号 | Mode | Profile |
|---|---|---|
| 用户想在后续编码过程中边做边学 | Teaching Card Mode | 默认 Intermediate，除非另有指定 |
| 用户询问已有改动、git diff 或最近 AI 修改 | Diff Walkthrough Mode | 默认 Intermediate，除非另有指定 |
| 用户要求解释代码片段、文件、函数、类或模块 | Code Explanation Mode | 默认 Intermediate，除非另有指定 |
| 用户说小白、零基础、非 CS、自学、要非常详细 | 当前 Mode | Novice |
| 用户说有一定基础、想看设计思路，或未指定水平 | 当前 Mode | Intermediate |
| 用户要求工程师视角、code review、架构、风险、可维护性或验证策略 | 当前 Mode | Engineer |

如果 Skill 已激活，但 Mode 和 Profile 仍不清楚，问一个简短澄清问题。

Profile 只是修饰器，不能单独激活 Skill。

## 5. Modes

### 5.1 Teaching Card Mode

当用户明确使用 `teach-as-you-code` 来覆盖后续编码过程时使用。

目的：

在 AI 编码过程中，解释有学习价值的关键步骤。

输出时机：

每完成一个有意义的学习单元后输出 Teaching Card，而不是每改一行都输出。

一个有意义的学习单元应该具备：

- 一个可理解的意图
- 一个可解释的技术选择
- 一个可验证的结果

例子：

- 新增组件
- 抽出 API helper
- 修复状态同步问题
- 添加表单校验
- 修改数据库 schema
- 添加回归测试

不要为这些内容输出 Teaching Card：

- 纯格式化
- import 排序
- 轻微文案修改
- 重复机械改动
- 没有学习价值的生成样板代码

### 5.2 Diff Walkthrough Mode

当用户明确使用 `teach-as-you-code` 来理解已有改动、diff 或最近 AI 修改时使用。

目的：

解释改了什么、为什么改、应该怎么读、以及如果从旧代码开始应该如何复现到当前版本。

默认包含以下内容，除非用户要求 Compact 输出，或信息确实不可用：

- 改动地图
- 阅读路径
- 复现路径
- 关键 Teaching Cards
- 跳过的机械改动
- 学习顺序
- 风险与验证

结构化 Diff Walkthrough 输出应在开头标明选中的 Mode 和 Audience Profile，例如：

```md
Mode: Diff Walkthrough
Profile: Intermediate
```

#### 阅读路径

回答这个问题：

```text
如果我要读懂这次改动，应该先看哪里，再看哪里？
```

不要按字母顺序或原始 diff 顺序列文件，而要按理解路径排序。

每个阅读路径步骤都应说明：

- 为什么这个文件或代码段排在这里
- 阅读时应该回答什么问题

常见模式：

- 前端：page/component entry -> hook/state -> API/client -> child components -> tests
- 后端：route/controller -> service/use case -> repository/model -> validation/errors -> tests
- 库/内部模块：public API -> core implementation -> helpers -> edge cases -> tests

#### 复现路径

回答这个问题：

```text
如果我从旧代码开始，要先改什么、再改什么、最后改什么，才能复现到现在？
```

Diff Walkthrough Mode 默认必须包含复现路径。它应该是按意图分组的实现顺序，不是逐行回放 diff。

每一步应包含：

- 改哪里
- 为什么这一步要在下一步之前
- 改完这一步后应该能看到什么
- 如果卡住，应该检查什么

除非用户要求 Compact 输出，或旧代码 / 新代码状态不足以可靠复原步骤，否则不要省略复现路径。

### 5.3 Code Explanation Mode

当用户明确使用 `teach-as-you-code` 来解释已有代码或粘贴代码时使用。

目的：

解释代码做什么、怎么运行、用了什么概念，以及存在什么风险或改造路径。

推荐格式：

```md
## Code Explanation: <short title>

### 1. 整体作用
这段代码负责什么。

### 2. 执行流程
控制流或数据流如何穿过这段代码。

### 3. 核心概念
涉及的重要 API、模式、语言特性或框架概念。

### 4. 关键代码讲解
解释最重要的部分，默认不逐行解释。

### 5. 注意点
bug、边界情况、隐藏假设或可读性问题。

### 6. 迁移或改造路径
这个思路如何复用、简化或演进。
```

如果代码很长，先概括结构，再选择最重要的区域深入解释。除非用户明确要求逐行讲解，否则不要逐行解释。

## 6. Audience Profiles

### 6.1 Novice Profile

面向真正的小白、非 CS 学习者、自学者，以及要求非常详细解释的用户。

风格：

- 概念优先
- 节奏更慢
- 明确解释术语
- 多用具体例子
- 默认包含小练习

重点：

- 代码解决什么问题
- 前置概念
- 数据如何流动
- 每个重要步骤为什么存在
- 常见错误
- 一个小练习

### 6.2 Intermediate Profile

默认档。

面向已经懂基本语法、能写简单功能，但想提升工程思维的用户。

风格：

- 推理优先
- 深度平衡
- 少讲基础语法
- 多讲数据流、状态、边界和取舍

重点：

- 改了什么或代码做什么
- 核心技术点
- 为什么这样设计
- 现实替代方案
- 当前取舍
- 如何迁移到其他场景

### 6.3 Engineer Profile

面向专业开发者、高级工程师、技术负责人，以及想要工程评审视角的用户。

风格：

- 取舍优先
- 简洁
- 关注架构
- 关注风险
- 关注验证

重点：

- 意图
- 职责边界
- 设计选择
- 取舍
- 风险面
- 验证策略
- 迁移、回滚或扩展路径

## 7. 输出格式

### 7.1 Teaching Card

```md
## Teaching Card: <short title>

### 1. 改了什么 / 这段代码做什么
解释具体改动或行为。

### 2. 用了什么技术
列出概念、API、模式或语言特性。

### 3. 为什么这样选
解释当前局部工程原因。

### 4. 底层原理
按当前 Profile 的深度解释技术如何工作。

### 5. 替代方案
列出现实替代方案和适用场景。

### 6. 为什么当前方案适合这里
结合当前项目约束解释取舍。

### 7. 迁移模式
说明以后在哪里能复用这个思路。

### 8. 关键记忆点
给出简短心智模型。

### 9. 小练习
有用时给出小练习；Novice 默认包含。
```

### 7.2 Diff Walkthrough Summary

```md
## Diff Walkthrough Summary: <short title>

Mode: Diff Walkthrough
Profile: <Novice | Intermediate | Engineer>

### 1. 目标
这次改动的高层目标。

### 2. 改动地图
按模块、层级或功能区分组。

### 3. 阅读路径
推荐阅读 diff 的顺序，并说明每一步为什么在这里、阅读时要回答什么问题。

### 4. 复现路径
推荐从旧代码复现到当前代码的顺序，并说明每一步改哪里、为什么下一步做它、改完后应该能看到什么。

### 5. 关键 Teaching Cards
精选高学习价值改动。

### 6. 跳过的机械改动
说明哪些低学习价值改动被摘要处理。

### 7. 学习顺序
先学什么、为什么。

### 8. 风险与验证
可能的回归和测试方式。
```

### 7.3 Code Explanation Summary

使用 Code Explanation Mode 的格式，并按 Profile 调整：

- Novice：包含前置概念、逐步流程、常见错误和练习。讲解应回答关键部分“这行/这段是什么意思”，但除非用户明确要求，不默认逐行解释。
- Intermediate：关注执行流程、设计意图、替代方案和迁移。默认用户懂基础语法，把篇幅放在各部分如何协作上。
- Engineer：关注边界、耦合、风险、可测试性和重构选项。输出更像技术评审，而不是入门教程。

推荐的 Profile-specific 结构：

```md
Novice:
### 这段代码想解决什么问题
### 你需要先知道的概念
### 按步骤看执行流程
### 关键行或关键片段
### 常见错误
### 小练习

Intermediate:
### 职责
### 执行流 / 数据流
### 关键设计选择
### 替代方案
### 迁移模式

Engineer:
### 职责边界
### 控制流 / 数据流
### 耦合与副作用
### 风险面
### 可测试性与重构选项
```

## 8. 大输入与 Token 预算

本 Skill 应主动控制输出长度。

### 8.1 建议长度预算

这些是实现建议，不是硬协议。

| 输出类型 | 目标长度（约 words） | 粗略 token 估算 |
|---|---:|---:|
| 单张 Teaching Card | 300-700 | 450-1,050 |
| Novice Teaching Card | 500-900 | 750-1,350 |
| Engineer Teaching Card | 250-600 | 375-900 |
| 中等分组总结 | 700-1,200 | 1,050-1,800 |
| 大 diff / 长代码总结 | 1,000-1,800 | 1,500-2,700 |
| 大输出中精选卡片数量 | 3-7 张卡片 | 适用于大 diff / 长代码场景 |

### 8.2 输出深度控制

第 8.1 的长度预算是默认 Standard 深度。Deep 或 Exhaustive 输出必须在 Skill 激活后由用户明确要求；普通的“小白”“解释一下”“教我”不会自动取消长度控制。

| 深度 | 激活后的示例触发词 | 行为 |
|---|---|---|
| Compact | “简短一点”“快速版”“只要摘要” | 压缩输出，只保留学习关键路径、风险和记忆点。 |
| Standard | 没有指定深度 | 使用 8.1 的建议预算。 |
| Deep | “非常详细”“讲深一点”“不要省略细节”“把原理讲清楚” | 允许更长解释，展开概念、数据流/控制流、设计理由、替代方案和迁移模式。 |
| Exhaustive | “逐行讲”“逐块讲”“全部讲清楚”“每个改动都走一遍” | 优先完整性而不是简短，但内容过大时应拆成多个 Part，而不是一次输出难以阅读的长墙。 |

Deep 和 Exhaustive 仍然是受控模式，不是无限输出。遇到很长的代码或 diff 时，先给目录或学习地图，再按编号 Part 分段继续。每个 Part 都应有清晰范围和停止点。

Deep 输出特别适合 Novice 学习者，但它独立于 Audience Profile。Novice 也可以是 Standard 长度，Engineer 也可以要求 Deep 架构分析。
### 8.3 大 diff 规则

满足以下情况时，不要为每个编辑点生成卡片，而要输出批量总结：

- diff 超过约 300 行
- 超过 6 个文件被修改
- 改动跨多个功能或重构
- 实现过程中没有渐进式输出卡片

大 diff 输出应：

- 先给改动地图
- 包含阅读路径和复现路径
- 精选 3-7 个高学习价值卡片
- 摘要机械改动
- 给出验证步骤

### 8.4 长代码规则

Code Explanation Mode 遇到长代码时：

- 先总结模块结构
- 找出入口、核心逻辑、副作用和可用测试
- 选择重要区域深入讲
- 除非用户明确要求，否则不逐行解释

### 8.5 长会话规则

长会话中：

- 编码过程中优先输出简洁卡片
- 用户要求时阶段性总结旧卡片
- 不重复已经讲过的内容，除非新决策需要
- 支持用户说“总结一下目前学到的东西”

## 9. 上下文、状态与降级策略

### 9.1 激活状态

默认只对当前请求生效，具体策略见“激活范围”。

实现建议：如果用户明确要求任务级、会话级或项目级范围，在可用的对话或项目指令上下文中保持激活状态。不要假设平台一定支持持久记忆。

### 9.2 git diff 降级

Diff Walkthrough Mode 应尽可能读取真实 diff。

如果 git diff 不可用：

- 使用可见的代码片段、文件内容、编辑器提供的改动或对话历史
- 明确说明限制
- 不要声称精确知道 before/after，除非确实可见
- 基于已有上下文提供 best-effort walkthrough

### 9.3 平台兼容

不要假设所有环境都有：

- git
- 文件系统访问
- 持久项目记忆
- 访问之前 AI 修改的能力

Skill 应优雅降级，并说明缺少哪些信息。

### 9.4 与其他 Skill 共存

`teach-as-you-code` 是解释层，不替代领域 Skill。

如果实际工作需要另一个 Skill，应先由那个 Skill 完成领域任务。只有当 `teach-as-you-code` 已明确激活时，`teach-as-you-code` 才作为外层教学包装，对结果、决策、代码或 diff 追加学习解释。

如果 `teach-as-you-code` 和另一个 Skill 被同时请求，领域 Skill 拥有执行优先级。`teach-as-you-code` 负责事后解释，不替代也不干扰领域工作流。

如果组合输出过长，优先完成领域任务，并提供紧凑学习摘要，而不是完整卡片。

## 10. 语言策略

默认语言跟随用户当前对话语言。

规则：

- 用户用中文，本 Skill 默认中文输出。
- 用户用英文，本 Skill 默认英文输出。
- 常见技术术语保留英文更清楚时，可以保留英文。
- 多语言示例中，必要时添加简短括号翻译。
- 不要把语言策略藏在某个 Profile 里。

## 11. 反模式

避免：

- 未明确激活 `teach-as-you-code` 就触发
- 默认逐行解释所有代码
- 为琐碎改动生成卡片
- 只说“提高可维护性”但不解释具体如何提高
- 假装只有一个正确方案
- 编造代码、diff 或对话中没有依据的动机
- 一次给小白塞太多概念
- 把一次性解释变成持续模式
- 未经明确偏好就永久把用户标记为小白或工程师

## 12. 推荐 SKILL.md Frontmatter

```yaml
---
name: teach-as-you-code
description: Use only when the user explicitly says "teach-as-you-code" or says "this Skill" in a context that clearly refers to teach-as-you-code. Provides structured learning explanations for code, AI-generated changes, and diffs after explicit activation. Never trigger for ordinary coding tasks, ordinary code explanations, ordinary diff reviews, general debugging questions, or standalone requests for teaching-card/profile-style explanations unless the user clearly requests teach-as-you-code or this Skill.
---
```
