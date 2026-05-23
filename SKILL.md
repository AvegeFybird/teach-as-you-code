---
name: teach-as-you-code
description: Use only when the user explicitly says "teach-as-you-code", "$teach-as-you-code", or says "this Skill" in a context that clearly refers to teach-as-you-code. Provides structured learning explanations for code, AI-generated changes, and diffs after explicit activation. Never trigger for ordinary coding tasks, ordinary code explanations, ordinary diff reviews, general debugging questions, or standalone requests for teaching-card/profile-style explanations unless the user clearly requests teach-as-you-code or this Skill.
---

# Teach As You Code

## Hard Activation Rule

Use this skill only when the current user request explicitly says `teach-as-you-code`, `$teach-as-you-code`, or says "this Skill" in a context that clearly refers to this skill.

If the user does not explicitly activate this skill, do not use its formats. Answer normally and concisely.

Do not activate for these requests by themselves:

- "Explain this code."
- "Review this diff."
- "Teach me while you code."
- "Use Teaching Card format."
- "I am a beginner; explain slowly."
- "Explain this from an engineer's perspective."

Activation is request-scoped by default. Continue across a task, session, or project only when the user explicitly asks for that scope. Stop when the user asks to turn off `teach-as-you-code`, stop using this skill, or just provide the result.

## Routing

After explicit activation, choose one Mode, one Audience Profile, and one Output Depth. In structured outputs, state the selected Mode, Profile, and Depth near the top unless the user asks for a very compact answer.

| Signal after activation | Mode | Default profile |
|---|---|---|
| User wants explanations during future coding steps | Teaching Card Mode | Intermediate |
| User asks about existing changes, a diff, or recent AI edits | Diff Walkthrough Mode | Intermediate |
| User asks to explain a snippet, file, function, class, or module | Code Explanation Mode | Intermediate |

Profile modifiers:

- Novice: user says beginner, zero-base, non-CS, self-taught, confused, or asks for a beginner-friendly explanation.
- Intermediate: user has some foundation, wants reasoning, or does not specify level.
- Engineer: user asks for engineering perspective, code review style, architecture, risk, maintainability, or verification.

Profiles cannot activate this skill by themselves.

Depth modifiers after activation:

- Compact: user asks for a short version, quick summary, or summary only.
- Standard: no depth modifier.
- Deep: user asks for a detailed, very detailed, slow, principle-oriented, or no-skipped-details explanation. In Chinese, requests such as "详细讲", "讲慢一点", "多讲一点", or "我是小白，详细讲" should route to Deep.
- Exhaustive: user asks for line-by-line, block-by-block, every change, or explain everything.

Depth modifiers cannot activate this skill by themselves.

## Modes

### Teaching Card Mode

Use for future coding work when the user explicitly activates `teach-as-you-code`.

Produce a Teaching Card after each meaningful learning unit, not after every line. A meaningful learning unit has one clear intent, one explainable technical choice, and one verifiable outcome.

If the user explicitly asks for a Teaching Card to explain a design choice or coding decision, you may produce one focused card even when no new edit is being made.

Good card triggers:

- adding a component
- extracting an API helper
- fixing state synchronization
- adding validation
- changing a schema
- adding a regression test

Skip cards for formatting-only edits, import sorting, trivial copy changes, repeated mechanical edits, or boilerplate with no learning value.

### Diff Walkthrough Mode

Use for existing changes, git diffs, or recent AI edits.

When possible, inspect the actual diff before explaining. If diff data is unavailable, state the limitation and use visible code, file contents, editor-provided changes, or conversation history. If the user provides a pseudo diff or natural-language change summary, explicitly say the explanation is based on that provided summary. Do not claim exact before/after differences unless they are visible.

Include these elements by default unless the user asks for Compact output or the information is unavailable:

- Change Map: group changes by module, layer, or feature area.
- Reading Path: where to start reading and what to read next.
- Rebuild Path: how to reproduce the current state from the old code.
- Key Teaching Cards: selected high-learning-value changes.
- Skipped Mechanical Changes: low-learning-value edits intentionally summarized.
- Learning Order: what to study first and why.
- Risks and Verification: likely regressions and how to test.

Reading Path answers: "If I want to understand this change, where should I start reading, and what should I read next?" Follow comprehension flow, not alphabetical or raw diff order. For each step, explain why this file or section comes here and include an explicit learner question. For Chinese output, label it as "思考问题"; for English output, label it as "Question to answer".

Examples: frontend entry -> state/hook -> API/client -> child components -> tests; backend route/controller -> service -> repository/model -> validation/errors -> tests.

Rebuild Path is required by default for Diff Walkthrough Mode, except in Compact output or when old/new state is not visible. It should be an implementation sequence grouped by intent, not a line-by-line replay. Each step should say where to edit, why it comes next, what should work after it, and what to check if stuck. Include an observable checkpoint or quick verification for each step when possible. For Chinese output, label it as "验证方式"; for English output, label it as "Checkpoint".

### Code Explanation Mode

Use for code snippets, files, functions, classes, or modules when the user explicitly activates `teach-as-you-code`.

Explain what the code does before discussing improvements. Follow execution flow instead of reading line by line by default. Do not invent surrounding project context when only a snippet is provided.

For long code, summarize the module structure first, identify entry points/core logic/side effects/tests if available, then select important regions for deeper explanation. Avoid line-by-line explanation unless requested. If Depth is Exhaustive, the explicit depth request overrides the default: explain line by line or block by block, and split into numbered parts when needed.

## Audience Profiles

### Novice

Use concept-first, slow-paced explanations with explicit terminology and concrete examples. Include a small exercise by default.

Emphasize prerequisite concepts, what problem the code solves, how data moves, why important steps exist, common mistakes, and one practice task.

### Intermediate

Use reasoning-first, balanced explanations. Assume basic syntax knowledge.

Emphasize what changed or what the code does, core technical ideas, why the design was chosen, realistic alternatives, tradeoffs, and transfer to other scenarios.

### Engineer

Use concise, tradeoff-first, architecture-aware explanations.

Emphasize intent, responsibility boundaries, design choices, coupling, risk surface, verification strategy, and migration/rollback/scaling paths.

## Output Formats

When referencing local files, prefer clickable absolute Markdown links with line numbers when the relevant line is known, such as `[core/routing.py](/F:/repo/core/routing.py:76)`. Use file-level links only when no precise line is visible.

### Teaching Card

```md
## 教学卡片：<short title>

Mode: Teaching Card
Profile: <Novice | Intermediate | Engineer>
Depth: <Compact | Standard | Deep | Exhaustive>

### 1. 改了什么 / 这段代码做什么
Explain the concrete code change or behavior.

### 2. 用了什么技术
Name the concepts, APIs, patterns, or language features involved.

### 3. 为什么这样做
Explain the local engineering reason for this choice.

### 4. 底层原理
Explain how the technique works at the right depth for the selected profile.

### 5. 还有哪些替代方案
List realistic alternatives and when they would fit.

### 6. 为什么这里适合
Tie the choice to the current project constraints, scale, style, or risk profile.

### 7. 如何迁移到其他场景
Explain where the learner can reuse the idea.

### 8. 关键 takeaway
Give a concise memory hook or 3-5 key points.

### 9. 小练习
Give a small exercise when useful; include one by default for Novice.
```

Compress sections when the user asks for brevity or when the profile does not need full detail. In Diff Walkthrough Mode, do not compress away Rebuild Path unless Compact output is requested or old/new state is not visible. For Novice, include practice by default and expand Teaching Cards enough to explain prerequisite concepts, fallback behavior, why the path matters, and how to transfer the idea. For Engineer, prefer compact review-style wording.

### Diff Walkthrough Summary

```md
## Diff Walkthrough 总结：<short title>

Mode: Diff Walkthrough
Profile: <Novice | Intermediate | Engineer>
Depth: <Compact | Standard | Deep | Exhaustive>

### 1. 改动目标
Explain the high-level goal of the change.

### 2. 改动地图
Group changes by module, layer, or feature area.

### 3. 阅读路径
Show the recommended order for reading the diff. For each step, include why it comes here and an explicit learner question. In Chinese output, format the question as `思考问题：...`.

### 4. 复现路径
Show the recommended order for reproducing the change from the old code. For each step, include where to edit, why this step comes next, what should work afterward, and a checkpoint or quick verification when possible. In Chinese output, format the checkpoint as `验证方式：...`.

### 5. 关键 Teaching Cards
Select high-learning-value changes and explain them deeply.
When embedding Teaching Cards inside this section, use `#### 教学卡片：<title>` / `#### Teaching Card: <title>` according to the output language, or use bold text. Do not use `## Teaching Card` because it breaks the parent summary hierarchy.

### 6. 跳过的机械改动
List low-learning-value edits intentionally summarized.

### 7. 建议学习顺序
Tell the learner what to study first and why.

### 8. 风险与验证
Explain possible regressions and how to test the change.
```

### Code Explanation Summary

```md
## 代码讲解：<short title>

Mode: Code Explanation
Profile: <Novice | Intermediate | Engineer>
Depth: <Compact | Standard | Deep | Exhaustive>

### 1. 整体作用
Explain what this code is responsible for.

### 2. 执行流程
Explain how control flow or data flow moves through the code.

### 3. 核心概念
Name important APIs, patterns, language features, or framework concepts.

### 4. 关键代码讲解
Explain the most important parts without explaining every line by default.

### 5. 注意点
Call out bugs, edge cases, hidden assumptions, or readability issues.

### 6. 迁移 / 重构路径
Explain how to reuse, simplify, or evolve this idea.
```

Profile-specific code explanation:

- Novice: add prerequisite concepts, step-by-step flow, key lines/sections, common mistakes, and practice.
- Intermediate: focus on responsibility, execution/data flow, key design choices, alternatives, and transfer.
- Engineer: focus on responsibility boundaries, control/data flow, coupling, side effects, risk, testability, and refactor options.

## Size Control

Control output size aggressively.

Suggested budgets:

- Single Teaching Card: 300-700 words.
- Novice Teaching Card: 500-900 words.
- Engineer Teaching Card: 250-600 words.
- Medium grouped summary: 700-1,200 words.
- Large diff/code summary: 1,000-1,800 words.
- Large outputs: select 3-7 key cards, not every edit.

Output depth:

- Compact: user asks for "shorter", "quick version", or "summary only". Compress output and keep only the learning-critical path, risks, and takeaways; in Diff Walkthrough Mode, summarize or omit Rebuild Path if it would make the answer too long.
- Standard: no depth modifier. Use the suggested budgets above.
- Deep: user explicitly asks for "detailed", "very detailed", "slow explanation", "deep dive", "do not skip details", or "explain the principles". Allow longer explanations and expand concepts, data/control flow, design rationale, alternatives, and transfer patterns. In Chinese, route "详细讲", "讲慢一点", "多讲一点", and "我是小白，详细讲" to Deep after explicit activation.
- Exhaustive: user explicitly asks for "line by line", "block by block", "explain everything", or "walk every change". Prioritize completeness, including line-by-line or block-by-block explanation when requested, but split large code or diffs into numbered parts instead of producing an unreadable wall of text.

Deep and Exhaustive must be explicitly requested after activation. Ordinary words such as "beginner", "explain", or "teach me" do not remove size controls unless paired with a clear detail/depth request.

Use a batch summary instead of one card per edit when a diff exceeds about 300 lines, more than 6 files changed, changes span multiple features/refactors, or cards were not emitted progressively during implementation.

In long sessions, prefer concise cards, do not repeat old explanations unless needed for a new decision, and support requests such as "summarize what I learned so far."

## Language

Follow the user's current conversation language by default. If the user is writing in Chinese, use Chinese for user-visible section headings and explanations, keeping only necessary protocol labels and technical terms in English, such as `Mode`, `Profile`, `Depth`, API names, class/function names, and widely used terms like `diff`, `route`, or `fallback`.

The output format examples below use Chinese headings as the default user-facing style. Use English headings only when the user's current conversation language is English or the user asks for English. Do not hide language choices inside a profile.

## Coexistence With Other Skills

`teach-as-you-code` is an explanation layer, not a replacement for domain skills.

If another skill is needed for the actual work, complete the domain skill's work first. Then use `teach-as-you-code` only as a teaching wrapper around the result, decision, or diff when this skill is explicitly active and the combined output remains useful.

If the combined output would become too long, prioritize completing the domain task and provide a compact learning summary.

## Anti-Patterns

Avoid:

- Do not invent motivations, design intent, or before/after behavior that is not supported by visible code, diff, or conversation.
- Do not permanently label the user as beginner, intermediate, or engineer without an explicit persistent preference.

## References

For full implementation details, read the matching specification only when needed:

- English: `TECHNICAL_SPEC.md`
- Chinese: `TECHNICAL_SPEC.zh-CN.md`
