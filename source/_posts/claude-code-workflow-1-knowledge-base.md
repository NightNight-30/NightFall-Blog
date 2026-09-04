---
title: Claude Code 工作流（一）：知识库双范式 — Obsidian × Notion
date: 2026-08-06 18:30:00
tags: [Claude Code, Obsidian, Notion, 知识库, 工作流]
topic: claude-code-tips
topic_order: 1
excerpt: 本地 markdown 双链 vs block + database + relation，附 daily-todo-bot 把 Notion 待办推钉钉的实战。Slidev deck 嵌入 + 逐页文字详解。
---

> 这是「Claude Code 工作流」系列第一篇，原文是内部分享 deck，星露谷物语像素风。下方嵌入了可交互的完整 deck，按方向键翻页，按 `O` 出缩略图多任务视图。本篇文字版对应 deck 的 Part 1（第 02-08 页）。

<iframe src="/claude-code-workflow-deck/?slide=1" width="100%" height="540" frameborder="0" style="border: 2px solid var(--wood-border, #b8a589); border-radius: 6px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); margin: 16px 0; max-width: 960px; aspect-ratio: 16/9;"></iframe>

> 移动端 iframe 体验有限，可[点此全屏查看 deck](/claude-code-workflow-deck/)。

## P02 · Part 1 目录：知识库工具

知识库这块我用两套工具：**Obsidian**（本地 markdown + 双链）和 **Notion**（block + database + relation）。这套对比不是为了 PK 谁强谁弱，而是搞清楚两套范式各自擅长什么、我各自在用它们做什么。

Part 1 共 6 节：1.1 范式对比 / 1.2 OB 在用 / 1.3 Notion 在用 / 1.4 daily-todo-bot / 1.5 优劣对比表 / 1.6 分工。

## P03 · 1.1 两种知识库范式对比

### Obsidian：本地 markdown + 双链

OB 的核心是**纯文本 `.md` 文件 + 本地存储**。文件系统就是数据库，`[[双链]]` 把文档织成网，graph view 看全局互连。跨 vault 用 Python 读写毫无障碍——它就是个文件夹。

**适合**：个人知识沉淀、离线写作、隐私敏感。
**痛点**：无原生 DB、无自动化触达、协作要迁整个 vault。

> "OB 织网 — 文档间互相关联"

### Notion：block + database + relation

Notion 的核心是 **block-based，无限嵌套**。block 是最小单位，paragraph / heading / toggle / callout 都是 block，可以拖拽组合。database + view + property 提供原生结构化能力，relation 把多个 DB 互相关联。云端，团队实时协作。

**适合**：结构化字段、多人协作、定时自动化。
**痛点**：离线不可用、导出受限、锁在云上。

> "Notion 垒层 — 内容分层可折叠"

一句话总结：**OB 织网，Notion 垒层**——前者文档间互相关联，后者内容内部分层。

## P04 · 1.2 Claude + OB：知识库使用方法

### 三个 vault

1. **项目变更记录** — 按天分区，`操作变更记录-YYYY-MM-DD.md`，原始数据层
2. **项目文档** — 按主题汇总，结构化层
3. **llm-wiki** — skill prompt 定义的三层结构（raw → wiki → index）

这套分层借鉴了数据开发思路：**原始 → 结构化 → 索引**。每日追加操作变更记录是原始层，项目文档按主题归档是结构化层，llm-wiki 是索引层。

### Claude 怎么读写

- `obsidian` skill — vault 读写。macOS 沙盒（TCC）限制 `~/Documents/` 直接访问，用 Python heredoc 绕开
- `karpathy-llm-wiki` — 三层结构维护
- `markitdown` — 网页/PDF/Word 转 md 再扔进 vault

### vault 结构示例

```
项目变更记录/
├── 操作变更记录/
│   └── 操作变更记录-2026-07-29.md
└── project/powerlink/
    ├── Claude Code 集成/
    │   └── 本地CLI与MCP连接方式对比.md
    ├── 数据接入/
    └── 数据字典/
```

### Claude 工作流

每日追加 → `obsidian` skill → 结构化归档。每日一个 md，跨 vault heredoc 批量检索。

## P05 · 1.3 Notion 当前在用

### 三个 database

1. **每日待办 DB** — agent 模板 5 字段：项目名 / 模块名 / 截止时间 / 四象限 / 任务内容。新增流程：关键词 → 给模板 → 填充提交 → 更新 Notion
2. **项目模块 DB** — 工作 + 个人两套库。PowerLink（工作）/ NightFall-Blog（个人），技术栈 / 上线日期 / 状态字段。relation：项目 → 子模块 → 每日待办（多对多）
3. **daily-todo-bot** — 9:00 / 18:00 推钉钉（详见下页）

### 配合的 MCP

`notion` MCP（mcp.notion.com/mcp，user scope）— 直连 Notion API，daily-todo-bot 自动化也走这条链路。

### 当前限制

- button 只读——MCP 暂不支持触发 Notion button
- select 受限——只能查不能改选项

## P06 · 1.4 daily-todo-bot：Notion 待办推到钉钉

### 做什么

每天 9:00 和 18:00 自动把今日待办从 Notion 拉出来，推送到企业钉钉的工作通知。上午推清单，下午推完成复盘。

### 链路

```
Notion 待办 DB
   ↓ query API（filter: 截止日期=today AND 状态≠已完成）
Python main.py
   ↓ corpcommunication asyncsend_v2
企业钉钉工作通知
```

### 三端协作

| 端 | 角色 | 关键 |
|---|---|---|
| ① Notion 端 | 数据源 | 每日待办 DB + Internal Integration token + filter |
| ② 中间发起 | Python main.py | cron `0 9,18 * * *` + 读 Notion + 调 `asyncsend_v2` |
| ③ 钉钉端 | 触达 | 企业内应用 + agent_id + appkey/secret 换 token + action_card |

### 部署

云服务器 `root@47.103.65.110:~/daily-todo-bot/`，cron 触发。

### 5 个开发坑

1. **TCC 限制** — 服务器调试，本地 macOS 沙盒绕开
2. **action_card** — 必带 `btn_json`，否则钉钉拒绝（400）
3. **钉钉日上限** — 频控限流，合并推送
4. **database_id ≠ data_source_id** — 查询要用 `collection://` 前缀
5. **create-pages** — 必须给 parent `data_source_id`，不能只给 `database_id`

> "Notion 是数据源，钉钉是触达层 — 中间一段 Python 把两者缝起来"

## P07 · 1.5 优劣对比表

| 维度 | Obsidian | Notion |
|---|---|---|
| 本地存储 | ✅ markdown 文件，可 grep | ❌ 云端，离线不可用 |
| 双链互关 | ✅ `[[` 一键织网 + graph view | ⚠️ relation 需手动建 |
| 层级嵌套 | ❌ 文档内部平铺 | ✅ 无限嵌套 + toggle 折叠 |
| 渲染层次 | ❌ 信息密度均匀 | ✅ 信息密度梯度分明 |
| 数据库能力 | ⚠️ 靠 frontmatter / dataview 插件 | ✅ 原生 database + view + filter |
| 团队协作 | ❌ 单人本地 | ✅ 多人实时 |
| 迁移成本 | ✅ 纯文本，可批量处理 | ❌ 锁在云上，导出有限 |
| 自动化触达 | ❌ 本地无定时能力 | ✅ 云端 API + cron 推钉钉 |
| 插件生态 | ✅ 海量社区插件，可自写 | ⚠️ 官方 API + 精选 integrations |
| 开源社区 | ✅ 完全开源，MIT 协议 | ❌ 闭源 SaaS，无法自托管 |

## P08 · 1.6 OB / Notion 分工：AI 读什么，人看什么

经过这轮对比，我的结论是**不二选一，分工合作**：

### OB — 喂 AI，本地知识源

纯文本 `.md` + 双链织网，Claude 跨 vault 用 Python heredoc 直接读写。

- **渐进式披露** — AI 按需读，不必一次拉全量
- **结构灵活** — 文件路径就是 schema
- **隐私本地** — 敏感记录不离机

> "OB 是 AI 的工作记忆 — 每日追加，跨 vault 检索"

### Notion — 喂人，项目追踪层

block + database + relation，感官直接，一路折叠到底看全貌。

- **感官直接** — 层次分明，按需展开
- **团队协作** — 多人实时，加人无需迁库
- **项目追踪** — 状态/日期/owner 字段一等公民

> "Notion 是人的仪表盘 — 一屏看全，一键深入"

### 联动：Notion 记录时参考 OB

更新 Notion 待办时，agent 可直接从 OB vault 读取历史记录/项目文档，结构化字段填入 DB — **一次写入，两端可用**。

---

下一篇：[Claude Code 工作流（二）：Claude Code × Databricks](/posts/claude-code-workflow-2-databricks/) — 远程 MCP Server vs 本地 CLI + Skills 选型，9 步测试工作流闭环，10 行 × 68 列自主抓取实战。

> 回到本专栏首页：[Claude Code 使用分享](/topic/claude-code-tips/)。
