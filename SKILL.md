---
name: context-handoff
description: "Use when the user wants to hand off the current conversation's context to another Claude Code session, or generate a context handoff / briefing document. Triggers: '上下文交接', '交接文档', 'handoff', '把上下文交给另一个会话', '迁移上下文', '生成交接文档', '总结上下文给新会话', '另一个会话要了解当前进度', 'context handoff', '让另一个 cwd 的会话了解这里'. Generates a structured markdown doc into the fixed folder ~/.claude/handoffs/."
---

# Context Handoff — 上下文交接文档生成

把当前会话的关键上下文蒸馏成一份 markdown 文档，供另一个会话（不同 cwd、不同机器）冷启动时接手。

这是跨 cwd 迁移上下文**最简单可靠**的方式——比搬 jsonl 文件（复制 + 换 sessionId + 改 cwd 字段）更稳，比官方 `SendMessage`（仅文本、Windows 不支持）更完整。

## 触发

用户说「交接上下文」「生成交接文档」「把当前进度交给另一个会话」「handoff」「迁移上下文」等时触发。

## 输出位置

固定输出到：

```
~/.claude/handoffs/
```

文件名：`<YYYY-MM-DD>_<topic-slug>.md`（日期 + 主题短横线小写 slug）。

> 若 `~/.claude` 尚未创建，先 `mkdir -p ~/.claude/handoffs`。
> 目录不存在时不要改写到别处——固定位置是这个 skill 可被检索的前提。

## 生成步骤

1. 回顾当前会话，识别真正值得交接的信息（见下方「写什么」）。
2. 按模板组织内容，用 Write 写到 `~/.claude/handoffs/<日期>_<topic>.md`。
3. 完成后告诉用户文件路径，并给一句可直接粘给新会话的指令（让新会话读该文件、先复述确认理解）。

## 写什么（信息选择）

只交接「新会话无法从代码 / git / CLAUDE.md 自行获得」的信息：

- **项目定位**：这是什么、核心设计理念、关键路径 / 命令
- **本次做了什么**：按逻辑列出，不是流水账
- **已核实的关键事实**：可直接复用的事实，标注 `[已验证]` / `[推理]` / `[不确定]`
- **当前结论与决策**：达成什么结论、为什么这么选、权衡是什么
- **待办 / 后续**：下一步、遗留问题、清理项

**不写**：完整逐字 transcript、思考块、每条工具调用；不写能从代码 / CLAUDE.md / `git log` 直接看出的东西。

## 模板

```markdown
# 上下文交接：<主题>

> 由 Claude Code 会话生成，交接给新会话继续。生成日期 <YYYY-MM-DD>。

## 一、项目 / 背景

<项目定位、核心设计理念、关键路径 / 命令>

## 二、本次会话做了什么

<按逻辑列出完成的工作>

## 三、已核实的关键事实

<可直接复用的事实，标注验证状态>

## 四、当前结论与决策

<结论、取舍、为什么>

## 五、待办 / 后续

<下一步、遗留、清理项>
```

## 原则

- 事实与推断分离：关键论断标注 `[已验证]` / `[推理]` / `[不确定]`。
- 简短优先：能 50 行说清就不写 200 行。
- 文档是给「冷启动的另一个会话」看的，要自包含——没有本会话上下文也能看懂。
- 文件名用日期 + 主题，方便在固定文件夹里排序和查找。
