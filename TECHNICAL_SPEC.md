# teach-as-you-code Technical Specification

Version 1.0 / 2026-05-18

## 1. Document Purpose

This document is a Skill specification for `teach-as-you-code`.

It is written for two audiences:

- implementers who want to build this Skill
- designers who want to reuse the design pattern in their own Skills

This is not only a product idea document. It is a prompt and behavior specification for a user-activated learning layer on top of AI-assisted coding.

It defines when the Skill intervenes, how it routes requests, how it structures output, how it controls length, and how it adapts to different learner levels.

## 2. Design Overview

`teach-as-you-code` helps users learn from AI-assisted coding without forcing every normal coding request into a long explanation.

The Skill has three core design ideas:

1. Strict opt-in activation
2. A three-dimensional output model: `Mode x Audience Profile x Output Depth`
3. Output safety controls for large diffs, long code, and long sessions

### 2.1 High-Level Flow

```text
User request
  -> Does it explicitly ask for teach-as-you-code or "this Skill"?
      -> No: answer normally; do not use this Skill.
      -> Yes: route by Mode, Audience Profile, and Output Depth.
  -> Determine Mode:
      -> future coding work: Teaching Card Mode
      -> existing changes/diff: Diff Walkthrough Mode
      -> code snippet/file/module: Code Explanation Mode
  -> Determine Audience Profile:
      -> beginner/non-CS/self-taught: Novice
      -> unspecified or some foundation: Intermediate
      -> engineering/review/risk/tradeoffs: Engineer
  -> Determine Output Depth:
      -> unspecified: Standard
      -> shorter / summary: Compact
      -> detailed / very detailed / slow explanation / deep dive: Deep
      -> line-by-line / explain everything: Exhaustive
  -> Determine input size:
      -> small: one focused card
      -> medium: grouped cards
      -> large: summary plus selected cards
  -> Produce structured explanation
  -> Accept in-context controls such as "shorter", "deep detail", "exhaustive line-by-line", "switch to engineer profile", or "turn off"
```

### 2.2 Three-Dimensional Output Model

```text
Mode = what situation is being explained.
Audience Profile = which learner level and perspective to use.
Output Depth = how detailed the output should be.
```

These three dimensions jointly determine the final answer. `Standard` is not a Profile; it is the default Output Depth.

Modes:

- Teaching Card Mode: explain meaningful coding steps while AI works.
- Diff Walkthrough Mode: explain existing changes, diffs, or recent AI modifications.
- Code Explanation Mode: explain user-provided code, files, functions, classes, or modules.

Audience Profiles:

- Novice: concept-first, terminology-explicit, beginner-friendly.
- Intermediate: reasoning-first, balanced depth, default profile.
- Engineer: tradeoff-first, risk-aware, architecture-oriented.

Output Depths:

- Compact: short version, keeping only the learning-critical path, risks, and takeaways.
- Standard: default depth, using the suggested length budgets.
- Deep: deeper explanation of principles, tradeoffs, data/control flow, and transfer patterns.
- Exhaustive: complete walkthrough for line-by-line, block-by-block, or highly detailed review; split large inputs into parts.

## 3. Activation Policy

The Skill is strictly opt-in.

Use this Skill only when the user explicitly says `teach-as-you-code` or says "this Skill" in a context that clearly refers to `teach-as-you-code`.

Do not activate the Skill for ordinary coding tasks, ordinary code explanations, ordinary diff reviews, general debugging questions, or standalone requests for Teaching Card/Profile-style explanation.

### 3.1 Activation Matrix

| User request | Activate Skill? | Reason |
|---|---:|---|
| "Explain this code." | No | Ordinary code explanation. |
| "I am a beginner; explain this code." | No | Audience signal only. |
| "Explain this from an engineer's perspective." | No | Audience signal only. |
| "Review this diff." | No | Ordinary diff review. |
| "Teach me while you code." | No | Teaching request, but not explicit Skill activation. |
| "Turn on Teaching Card Mode." | No | Named format, but not explicit Skill activation. |
| "Use Teaching Card format." | No | Named format, but not explicit Skill activation. |
| "Use teach-as-you-code to explain this code." | Yes | Explicit Skill request. |
| "Use this Skill to explain this code." | Yes | Explicit if current context refers to this Skill. |
| "Use teach-as-you-code Diff Walkthrough Mode." | Yes | Explicit Skill + mode request. |
| "Use teach-as-you-code novice profile." | Yes | Explicit Skill + profile request. |
| "Use teach-as-you-code engineer profile to review this diff." | Yes | Explicit Skill + profile + mode request. |

### 3.2 Activation Scope

Default scope is the current request.

The Skill should continue across multiple steps only if the user explicitly sets a wider scope:

- current feature/task
- current conversation
- current project, if persistent project instructions are supported

Stop when the user says:

- "turn off teach-as-you-code"
- "stop using this Skill"
- "just give me the result"
- "no Teaching Cards for now"

## 4. Routing Rules After Activation

Once the Skill is explicitly activated, route the request using this table.

| Signal after activation | Mode | Profile |
|---|---|---|
| User wants explanations during future coding steps | Teaching Card Mode | Default Intermediate unless specified |
| User asks about existing changes, a git diff, or recent AI edits | Diff Walkthrough Mode | Default Intermediate unless specified |
| User asks to explain a code snippet, file, function, class, or module | Code Explanation Mode | Default Intermediate unless specified |
| User says beginner, zero-base, non-CS, self-taught, or asks for beginner-friendly explanation | Current Mode | Novice |
| User says they have some foundation, wants design reasoning, or does not specify level | Current Mode | Intermediate |
| User asks for engineer perspective, code review style, architecture, risk, maintainability, or verification | Current Mode | Engineer |

Then choose Output Depth separately:

| Signal after activation | Depth |
|---|---|
| No depth modifier | Standard |
| User asks for shorter, quick version, or summary only | Compact |
| User asks for detailed, very detailed, slow, no-skipped-details, or principle-oriented explanation | Deep |
| User asks for line-by-line, block-by-block, every change, or explain everything | Exhaustive |

If both Mode and Profile are unclear after activation, ask one short clarification question.

Profiles are modifiers. They cannot activate the Skill by themselves.

## 5. Modes

### 5.1 Teaching Card Mode

Use when the user explicitly activates `teach-as-you-code` for future coding work.

Purpose:

Explain meaningful AI coding steps as they happen.

Output timing:

Produce a Teaching Card after each meaningful learning unit, not after every line or tiny edit.

A meaningful learning unit is a small, coherent coding action with:

- one understandable intent
- one explainable technical choice
- one verifiable outcome

Examples:

- adding a component
- extracting an API helper
- fixing state synchronization
- adding validation
- changing a database schema
- adding a regression test

Do not produce Teaching Cards for:

- formatting-only edits
- import sorting
- trivial copy changes
- repeated mechanical edits
- generated boilerplate with no learning value

### 5.2 Diff Walkthrough Mode

Use when the user explicitly activates `teach-as-you-code` to understand existing changes, a diff, or recent AI edits.

Purpose:

Explain what changed, why it changed, how to read the change, and how to reproduce it from the previous code.

Default elements, unless the user asks for Compact output or the information is unavailable:

- change map
- Reading Path
- Rebuild Path
- key Teaching Cards
- Skipped Mechanical Changes
- Learning Order
- risks and verification

Structured Diff Walkthrough outputs should state the selected Mode and Audience Profile near the top, for example:

```md
Mode: Diff Walkthrough
Profile: Intermediate
```

#### Reading Path

Answers:

```text
If I want to understand this change, where should I start reading, and what should I read next?
```

Order files by comprehension flow, not alphabetically or by raw diff order.

Each Reading Path step should explain:

- why this file or section comes at this point
- what question the learner should answer while reading it

Common patterns:

- Frontend: page/component entry -> hook/state -> API/client -> child components -> tests
- Backend: route/controller -> service/use case -> repository/model -> validation/errors -> tests
- Library: public API -> core implementation -> helpers -> edge cases -> tests

#### Rebuild Path

Answers:

```text
If I started from the old code, what should I change first, second, and third to reproduce the current result?
```

Rebuild Path is required by default for Diff Walkthrough Mode. It should be an implementation sequence grouped by intent, not a line-by-line replay.

Each step should include:

- where to edit
- why this step comes before the next one
- what should work after this step
- what to check if the user gets stuck

Do not omit Rebuild Path unless the user asks for Compact output or the old/new state is not visible enough to reconstruct a reliable sequence.

### 5.3 Code Explanation Mode

Use when the user explicitly activates `teach-as-you-code` to explain code that already exists or code they pasted.

Purpose:

Explain what the code does, how it runs, what concepts it uses, and what risks or improvement paths exist.

Recommended format:

```md
## Code Explanation: <short title>

### 1. Overall Purpose
What this code is responsible for.

### 2. Execution Flow
How control flow or data flow moves through the code.

### 3. Core Concepts
Important APIs, patterns, language features, or framework concepts.

### 4. Key Code Walkthrough
The important parts, without explaining every line by default.

### 5. Watchouts
Bugs, edge cases, hidden assumptions, or readability issues.

### 6. Transfer or Refactor Path
How to reuse, simplify, or evolve this idea.
```

For long code, summarize structure first, then select the most important regions to explain. Do not explain every line unless the user explicitly asks for a line-by-line walkthrough.

## 6. Audience Profiles

### 6.1 Novice Profile

For true beginners, non-CS learners, self-taught learners, and users who ask for beginner-friendly explanations.

Style:

- concept-first
- slow pacing
- explicit terminology
- concrete examples
- small exercises by default

Emphasize:

- what problem the code solves
- prerequisite concepts
- how data moves through the code
- why each important step exists
- common mistakes
- one small practice task

### 6.2 Intermediate Profile

Default profile.

For users who know basic syntax and can build simple features, but want stronger engineering reasoning.

Style:

- reasoning-first
- balanced depth
- less syntax explanation
- more data flow, state, boundaries, and tradeoffs

Emphasize:

- what changed or what the code does
- core technical ideas
- why this design was chosen
- realistic alternatives
- tradeoffs
- transfer to other scenarios

### 6.3 Engineer Profile

For professional developers, senior engineers, tech leads, and users who want an engineering review perspective.

Style:

- tradeoff-first
- concise
- architecture-aware
- risk-aware
- verification-focused

Emphasize:

- intent
- responsibility boundaries
- design choices
- tradeoffs
- risk surface
- verification strategy
- migration, rollback, or scaling path

## 7. Output Formats

### 7.1 Teaching Card

```md
## Teaching Card: <short title>

### 1. What Changed / What This Does
Explain the concrete code change or behavior.

### 2. Techniques Used
Name the concepts, APIs, patterns, or language features.

### 3. Why This Approach
Explain the local engineering reason.

### 4. Underlying Principle
Explain how the technique works at the right depth for the profile.

### 5. Alternatives
List realistic alternatives and when they fit.

### 6. Why This Fits Here
Tie the choice to the current project constraints.

### 7. Transfer Pattern
Explain where the learner can reuse the idea.

### 8. Key Takeaways
Give a concise memory hook.

### 9. Practice
Give a small exercise when useful, required by default for Novice.
```

### 7.2 Diff Walkthrough Summary

```md
## Diff Walkthrough Summary: <short title>

Mode: Diff Walkthrough
Profile: <Novice | Intermediate | Engineer>

### 1. Goal
High-level goal of the change.

### 2. Change Map
Group changes by module, layer, or feature area.

### 3. Reading Path
Recommended order for reading the diff, including why each step comes there and what question to ask while reading.

### 4. Rebuild Path
Recommended order for reproducing the change from old code, including where to edit, why this step comes next, and what should work afterward.

### 5. Key Teaching Cards
Selected high-learning-value changes.

### 6. Skipped Mechanical Changes
Low-learning-value edits that are intentionally summarized.

### 7. Learning Order
What to study first and why.

### 8. Risks and Verification
Possible regressions and how to test.
```

### 7.3 Code Explanation Summary

Use the format from Code Explanation Mode. Adapt depth by profile:

- Novice: include prerequisite concepts, step-by-step flow, common mistakes, and practice. The explanation should answer "what does this line/section mean?" for the most important parts, but still avoid explaining every line unless requested.
- Intermediate: focus on execution flow, design intent, alternatives, and transfer. The explanation should assume basic syntax knowledge and spend most of its detail on how the parts work together.
- Engineer: focus on boundaries, coupling, risk, testability, and refactor options. The explanation should read closer to a technical review than a tutorial.

Recommended profile-specific shapes:

```md
Novice:
### What this code is trying to do
### Concepts you need first
### Step-by-step flow
### Key lines or sections
### Common mistakes
### Small practice task

Intermediate:
### Responsibility
### Execution/Data Flow
### Key Design Choices
### Alternatives
### Transfer Pattern

Engineer:
### Responsibility Boundary
### Control/Data Flow
### Coupling and Side Effects
### Risk Surface
### Testability and Refactor Options
```

## 8. Large Input and Token Budget

The Skill should control output size aggressively.

### 8.1 Suggested Length Budgets

These are implementation guidelines, not hard protocol limits.

| Output type | Target size (approx words) | Rough token estimate |
|---|---:|---:|
| Single Teaching Card | 300-700 | 450-1,050 |
| Novice Teaching Card | 500-900 | 750-1,350 |
| Engineer Teaching Card | 250-600 | 375-900 |
| Medium grouped summary | 700-1,200 | 1,050-1,800 |
| Large diff/code summary | 1,000-1,800 | 1,500-2,700 |
| Selected key cards in large output | 3-7 cards | Applies to large diff/code cases |

### 8.2 Output Depth Controls

Length budgets are the default Standard depth. Deep or exhaustive output must be explicitly requested after activation; ordinary words such as "beginner", "explain", or "teach me" do not remove size controls unless paired with a clear detail/depth request.

| Depth | Example triggers after activation | Behavior |
|---|---|---|
| Compact | "shorter", "quick version", "summary only" | Compress output, keep only the learning-critical path, risks, and takeaways. |
| Standard | no depth modifier | Use the suggested budgets in 8.1. |
| Deep | "detailed", "very detailed", "slow explanation", "deep dive", "do not skip details", "explain the principles", "I am a beginner; explain in detail" | Allow longer explanations, expand concepts, data/control flow, design rationale, alternatives, and transfer patterns. |
| Exhaustive | "line by line", "block by block", "explain everything", "walk every change" | Prioritize completeness over brevity, but split into parts when needed instead of producing an unreadable wall of text. |

Deep and Exhaustive are still controlled modes, not unlimited output. For very large code or diffs, first provide a table of contents or learning map, then continue in numbered parts. Each part should have a clear scope and stopping point.

Deep output is especially useful for Novice learners, but it is independent from Audience Profile. A Novice explanation can still be Standard length, and an Engineer explanation can still request Deep architectural analysis.
### 8.3 Large Diff Rules

Use a batch summary instead of one card per edit when:

- diff exceeds about 300 lines
- more than 6 files changed
- the change spans multiple features or refactors
- the assistant did not emit cards progressively during implementation

For large diffs:

- start with a change map
- include Reading Path and Rebuild Path
- select 3-7 high-learning-value cards
- summarize mechanical changes
- give verification steps

### 8.4 Long Code Rules

For long code in Code Explanation Mode:

- summarize the module structure first
- identify entry points, core logic, side effects, and tests if available
- select important regions for deeper explanation
- avoid line-by-line explanation unless requested

### 8.5 Long Session Rules

For long sessions:

- prefer concise cards while coding
- periodically summarize previous cards if the user asks
- do not repeat old explanations unless they are needed for a new decision
- allow the user to say "summarize what I learned so far"

## 9. Context, State, and Fallbacks

### 9.1 Activation State

Activation is request-scoped by default, as defined in Activation Scope.

Implementation guidance: if the user explicitly asks for task/session/project scope, keep the active state in the available conversation or project instruction context. Do not assume persistent memory exists unless the platform provides it.

### 9.2 git diff Fallbacks

Diff Walkthrough Mode should inspect actual diffs when possible.

If git diff is unavailable:

- use the code snippets, file contents, editor-provided changes, or conversation history available
- state the limitation clearly
- avoid claiming exact before/after differences unless they are visible
- offer a best-effort walkthrough based on available context

### 9.3 Platform Compatibility

Do not assume every environment has:

- git
- file system access
- persistent project memory
- access to previous AI edits

The Skill should degrade gracefully and explain what information is missing.

### 9.4 Coexistence With Other Skills

`teach-as-you-code` is an explanation layer, not a replacement for domain Skills.

If another Skill is needed for the actual work, that Skill should complete the domain task first. `teach-as-you-code` should then act only as an outer teaching wrapper around the result, decision, code, or diff when this Skill is already explicitly active.

If `teach-as-you-code` and another Skill are both requested, the domain Skill has execution priority. `teach-as-you-code` explains the work afterward; it should not replace or interfere with the domain workflow.

If the combined output would become too long, prioritize completing the domain task and provide a compact learning summary instead of full cards.

## 10. Language Policy

Default language follows the user's current conversation language.

Rules:

- If the user writes in Chinese, answer in Chinese by default.
- If the user writes in English, answer in English by default.
- Keep common technical terms in English when that is clearer.
- If examples are multilingual, add short parenthetical translations where useful.
- Do not hide language policy inside a specific Audience Profile.

## 11. Anti-Patterns

Avoid:

- triggering without explicit `teach-as-you-code` activation
- explaining every line by default
- producing cards for trivial edits
- using generic phrases like "improves maintainability" without explaining how
- pretending there is only one correct solution
- inventing motivations not supported by code, diff, or conversation
- overwhelming beginners with too many concepts at once
- turning a one-time explanation into a persistent mode
- permanently labeling the user as beginner or engineer without explicit preference

## 12. Recommended SKILL.md Frontmatter

```yaml
---
name: teach-as-you-code
description: Use only when the user explicitly says "teach-as-you-code" or says "this Skill" in a context that clearly refers to teach-as-you-code. Provides structured learning explanations for code, AI-generated changes, and diffs after explicit activation. Never trigger for ordinary coding tasks, ordinary code explanations, ordinary diff reviews, general debugging questions, or standalone requests for teaching-card/profile-style explanations unless the user clearly requests teach-as-you-code or this Skill.
---
```
