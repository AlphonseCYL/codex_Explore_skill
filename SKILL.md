---
name: explore
description: Use when a coding task requires broad codebase reconnaissance before implementation, especially unfamiliar repositories, architecture questions, feature tracing, bug investigations, refactors, migrations, reviews, or changes that may require reading roughly 10+ files or searching multiple independent areas. Delegates read-only exploration to explorer subagents first, keeps the main conversation lean, prevents the main agent from reading code while subagents are exploring, and requires a key-files table before the main agent proceeds with detailed code reading or edits. Works naturally in Chinese conversations while preserving Codex-facing English formats.
---

# Explore

## Core Rule

在主对话里阅读很多文件之前，先把代码库侦察分派给一个或多个 explorer 子代理。主上下文只负责协调、决策、实现和验证；让子代理消化大量搜索结果和文件读取内容。

当出现以下信号时，使用这个 skill：

- 任务位于不熟悉的仓库或子系统中。
- 任务大概率需要阅读约 10 个或更多文件。
- 任务跨越多个独立区域，例如 API、数据库、UI、测试、文档或构建工具。
- 用户要求理解架构、追踪功能、重构、迁移、分析 bug 根因，或做大范围评审。
- 在编辑前需要一次新的、只读的摸底。

如果只是一个很小、目标明确的改动，而且相关文件已经明确，就可以跳过分派。

## Workflow

1. 简短说明你正在使用 explorer 子代理做侦察。
2. 识别彼此独立的研究切片；代码库较大时优先拆成 2 到 4 个切片。
3. 在环境支持子代理且策略允许时，创建 explorer 子代理。给每个子代理一个窄而明确的只读任务。默认将 explorer 的 reasoning effort 设为 `low`；只有在任务在架构上复杂、含糊、高风险，或存在多条相似且容易混淆的代码路径时，才提升到 `medium` 或 `high`。
4. 子代理运行时，主对话不要读取目标代码库中的代码。只做协调、等待，或处理与该代码库无关的事情。
5. 收集 explorer 的输出，并在自己继续读大文件之前先综合结果。
6. 先产出 key-files 表，再把它作为下一步阅读地图。

## Explorer Prompt Template

给每个 explorer 子代理都使用下面的英文结构。中文语境下，可以让 explorer 用中文写发现、理由和说明，但不要翻译 `Goal`、`Scope`、`Find`、`Return only`、表格列名等格式字段。

```text
You are doing read-only codebase reconnaissance. Do not edit files.

Goal: <specific research question>
Scope: <directories, modules, packages, feature names, or search terms>

Find:
- The most relevant files and symbols.
- How the pieces fit together.
- Which code paths are active, primary, legacy, experimental, or unused when that can be inferred.
- Existing patterns the main agent should preserve.
- Tests, fixtures, configs, or docs that matter.
- How likely changes should be grouped by concern, such as UI, API, state, data model, background jobs, permissions, i18n, styling, tests, or configuration.
- Risks, unclear areas, and follow-up reads.

Return only a concise report with:
1. Findings
2. Key files table
3. Suggested next reads

Key files table columns:
| File | Why it matters | Relevant symbols or sections | Confidence |
```

如果用户使用中文，或目标仓库的文档/注释以中文为主，explorer 的 `Findings` 和表格内容应优先用中文表达；上面的标题、编号和表格列名仍保持英文。

## Slicing Guidance

按自然边界拆分：

- 垂直功能链路：前端、API 路由、服务层、持久化、测试。
- 分层架构：UI、领域逻辑、数据模型、集成、构建/运行时配置。
- 问题假设：可能的根因区域、复现路径、相邻实现。
- 单体仓库中的独立包。

避免使用“把一切都探索一遍”这类含糊提示。一个好的 explorer 任务应当同时具备范围、问题和期望输出格式。

## Required Synthesis

子代理完成后，在大范围阅读或修改之前，先总结有用发现并包含下表：

| File | Role | Read next? | Reason | Source |
| --- | --- | --- | --- | --- |
| `path/to/file` | 入口 / 模型 / 测试 / 配置 / helper | Yes / Maybe / No | 为什么这个文件会影响任务 | explorer 名称或本地验证 |

用这张表决定接下来本地先读什么。优先阅读标记为 `Yes` 的文件。

当用户在问功能应该改哪里时，再补一张简短的 “Change Map”：

| Change type | Primary files | Notes |
| --- | --- | --- |
| UI / API / data model / tests / config | `path/to/file` | 这里该改什么，以及要避免什么 |

当存在多个相似实现时，要明确标注每个实现是 `primary`、`legacy`、`experimental`、`generated` 还是 `unclear`。并说明支撑该判断的证据，例如路由使用情况、API 客户端使用情况、测试、文档、命名，或最近的上下文模式。

## Guardrails

- 把 explorer 的工作视为侦察，不是实现。
- 向 explorer 询问摘要和文件引用，不要要完整文件转储。
- 除非用户明确要求委派编辑且环境允许，否则 explorer 提示词都保持只读。
- 只有在切片彼此独立时，才并行使用多个 explorer。
- 在 explorer 子代理阅读目标代码库期间，主代理不得对同一目标执行本地代码搜索、文件读取或结构扫描。
- 如果子代理不可用，就用本地 `rg`、`rg --files` 和简短笔记执行同样的流程，但仍然要产出 key-files 表。
- 如果 explorer 发现彼此冲突，先在本地验证最小相关文件集，再继续修改。
- 不要把某次调查的项目特定结论写进这个 skill 本身；这里只保留可复用的规则和输出形状。

## Design Notes

这个工作流把 Claude Code 的 Explore Agent 模式适配到了 Codex：把大规模代码搜索隔离到独立上下文，只返回相关摘要，并通过 key-files 表把交接做实。要查看来源说明，请在更新这个 skill 或解释其设计时阅读 `references/claude-explore-agent-research.md`。
