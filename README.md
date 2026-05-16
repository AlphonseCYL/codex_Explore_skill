# Explore

Explore 是一个给 Codex 使用的探索型 skill，用于在开始修改代码之前，先对代码库做一轮大范围侦察。

它适合这些场景：

- 进入一个不熟悉的仓库
- 需要理解架构、模块关系或数据流
- 需要追踪某个功能是怎么实现的
- 需要做 bug 根因分析、重构、迁移或评审
- 预计要阅读很多文件，或要同时看多个独立区域

这个 skill 的目标不是直接修改代码，而是先帮你把“该看什么、先看什么、哪些文件最关键”理清楚。

## 使用方法

把这个仓库放到你的 Codex skills 目录后，直接在对话里使用：

```text
Use $explore to map this codebase before making changes.
```

在中文语境下，也可以这样理解它的作用：

```text
先用 $explore 理清这个代码库，再开始改动。
```

## 工作方式

这个 skill 会引导 Codex：

1. 先把大范围代码侦察交给 explorer 子代理
2. 子代理只做只读阅读，不修改文件
3. 让子代理返回简洁发现、key-files 表和建议的下一步阅读
4. 主代理根据这些结果，再决定具体怎么读代码、怎么改动

这样可以避免一上来就把主上下文塞满，也能减少漏看关键文件的概率。

## 它要求什么

- 先做只读探索，再做修改
- 主代理在子代理探索期间不要抢先读很多目标文件
- 先输出 key-files 表，再进入深入阅读
- 尽量区分 primary、legacy、experimental、generated、unused、unclear 等代码路径
- 按 UI、API、data model、tests、config 等关注点整理可能的改动

## 目录说明

- `SKILL.md`：这个 skill 的核心规则和工作流
- `agents/openai.yaml`：展示名称、简短描述和默认提示词
- `references/`：设计参考和来源说明
- `.gitignore`：Git 忽略规则

## 备注

这个 skill 已经按中文使用场景做了本地化，但保留了 Codex 依赖的英文格式字段，例如 `name`、`description`、`Goal`、`Scope` 和表格列名。

