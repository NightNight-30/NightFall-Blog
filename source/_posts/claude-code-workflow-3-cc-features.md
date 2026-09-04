---
title: Claude Code 工作流（三）：内置功能 — agents view + /insights
date: 2026-08-06 19:00:00
tags: [Claude Code, agents view, insights, 后台会话, 工作流]
topic: claude-code-tips
topic_order: 3
excerpt: 一屏管理所有后台会话的状态体系（六状态 + 三形状），历史会话分析报告的正确用法与常见误区。Slidev deck 嵌入 + 逐页文字详解。
---

> 这是「Claude Code 工作流」系列第三篇，也是最后一篇，对应 deck 的 Part 3（第 13-16 页）。下方嵌入了可交互的完整 deck，按方向键翻页，按 `O` 出缩略图多任务视图。

<iframe src="/claude-code-workflow-deck/?slide=12" width="100%" height="540" frameborder="0" style="border: 2px solid var(--wood-border, #b8a589); border-radius: 6px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); margin: 16px 0; max-width: 960px; aspect-ratio: 16/9;"></iframe>

> 移动端 iframe 体验有限，可[点此全屏查看 deck](/claude-code-workflow-deck/)。

## P13 · Part 3 目录：Claude Code 内置功能

Part 3 共 2 节：3.1 agents view + 3.2 /insights。这两个是 Claude Code 自带能力，不需要额外装 skill 或 MCP，但很多人要么没用过，要么用错了。

## P14 · 3.1 agents view：一屏管所有后台会话

### 是什么

`claude agents` 命令开一个总览屏，一屏看到所有 Claude Code 会话：在跑的、等你输入的、已完成的。任意一个都能 `/resume` 恢复，上下文完整保留。

### 解决的痛点

- 以前：关掉终端 = 会话没了，上下文丢
- 以前：同时跑多个任务没法切换，得开多个终端窗口手动记
- 以前：想知道「那个长任务跑完了没」只能回去翻历史
- 现在：一屏总览 + 一键 `/resume` + 状态实时可见

### 六种状态（图标颜色 = 会话状态）

| 图标 | 状态 | 含义 |
|---|---|---|
| 🔵 动效中 | **Working** | 正在跑工具/生成 |
| 🟡 黄 | **Needs input** | 等你回答/授权 |
| ⚫ 暗淡 | **Idle** | 空闲待命 |
| 🟢 绿 | **Completed** | 完成 |
| 🔴 红 | **Failed** | 报错 |
| ⚪ 灰 | **Stopped** | 被 Ctrl+X / `claude stop` 终止 |

### 三种形状（图标形状 = 进程是否存活）

颜色独立于形状，形状表示进程状态——这是官网文档里容易忽略的点：

| 形状 | 含义 |
|---|---|
| ✻ | 进程存活，立即响应 |
| ∙ | 进程已退出，仍可 peek/reply，从断点续 |
| ✢ | `/loop` 迭代间休眠，显示运行次数+倒计时 |

### 顺带一提：/goal + /bg 组合

- `/goal` — 设一个条件，Claude 跨多轮自主拆解、持续推进直到条件达成
- `/bg` — 把当前会话切到后台继续跑，释放终端
- **组合用法**：`/goal` 设目标 + `/bg` 后台跑 = 长任务无人值守推进，在 agents view 里看进度

参考：[code.claude.com/docs/en/agent-view](https://code.claude.com/docs/en/agent-view)

## P15 · 3.2 /insights 正确用法

### 是什么

`/insights` 生成一份分析 Claude Code 会话的报告。覆盖：项目分布、交互模式、摩擦点（friction points）。读的是 `~/.claude/projects` 下的历史记录。输出是一个 HTML 文件，存在当前工作目录。

### 为什么用

- 看消耗在哪
- 识别浪费的 token / cache / subagent 调用
- 纠正低效的 prompt 模式

### 常见误区

- ❌ **不是实时遥测** — 读历史记录，不是监控当前会话
- ❌ **不是聊天总结** — 生成 HTML 文件，不是返回聊天里的总结
- ❌ **依赖会话记录** — 空目录无输出

### 正确姿势

```
/insights 7d
→ 看报告里的异常发现
→ 调整 prompt / 拆分 skill / 优化 subagent 调用
```

### 典型发现 → 优化方向

- **skill 反复调 40% token** → 合并步骤
- **subagent 重读同一文件** → 加 memory 缓存
- **长 prompt 拆短 cache ↑** → 拆 system/user 块利用 prompt cache

参考：[code.claude.com/docs/en/commands](https://code.claude.com/docs/en/commands)

> `/insights` 不是看一眼就完——要落地改 prompt，不然下次还是同样的浪费。

## P16 · Thanks

> Q & A 时间
> Obsidian × Notion × Databricks × Claude Code

系列三篇到此结束。三篇都配了可交互的 deck 嵌入，翻一翻 deck 看视觉，读一读文字看细节。

---

> 回到本专栏首页：[Claude Code 使用分享](/topic/claude-code-tips/)。
> 系列导航：[第一篇 知识库](/posts/claude-code-workflow-1-knowledge-base/) · [第二篇 Databricks](/posts/claude-code-workflow-2-databricks/) · 当前篇
