---
title: Claude Code 工作流（二）：Claude Code × Databricks
date: 2026-08-06 18:45:00
tags: [Claude Code, Databricks, MCP, Skills, 工作流]
topic: claude-code-tips
topic_order: 2
excerpt: 远程 MCP Server vs 本地 CLI + Skills 选型，9 步测试工作流闭环，10 行 × 68 列自主抓取实战。Slidev deck 嵌入 + 逐页文字详解。
---

> 这是「Claude Code 工作流」系列第二篇，对应 deck 的 Part 2（第 09-12 页）。下方嵌入了可交互的完整 deck，按方向键翻页，按 `O` 出缩略图多任务视图。

<iframe src="/claude-code-workflow-deck/?slide=8" width="100%" height="540" frameborder="0" style="border: 2px solid var(--wood-border, #b8a589); border-radius: 6px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); margin: 16px 0; max-width: 960px; aspect-ratio: 16/9;"></iframe>

> 移动端 iframe 体验有限，可[点此全屏查看 deck](/claude-code-workflow-deck/)。

## P09 · Part 2 目录：Claude Code × Databricks

Part 2 共 3 节：2.1 两种连接方式对比 + 选型 / 2.2 测试工作流（Jobs 模式）/ 2.3 实战示例。

核心问题：**Claude Code 怎么连 Databricks**？答案是——**不要走第三方 MCP Server，直接用官方 CLI + markdown Skills**。

## P10 · 2.1 两种连接方式对比 + 选型结论

### 方式一：远程 MCP Server

架构：`Claude ⇄ MCP 协议 ⇄ 第三方 Server ⇄ Databricks API`

- 依赖：找/装第三方 Server
- 能力：受限于 Server 暴露的工具
- 拓展：改 Server 源码重启
- Azure 中国：看社区实现，不一定可用

### 方式二：本地 CLI + Skills ✓（选型结论）

架构：`Claude ⇄ Bash ⇄ databricks CLI ⇄ Databricks API`

- 依赖：官方 CLI + markdown Skills
- 能力：全 CLI 子命令 + REST
- 拓展：直接调子命令
- Azure 中国：官方原生支持

### 配合的 skill

- `databricks-core` — 日常 CLI 操作、auth、profile、bundle
- `databricks-pipelines` — Lakeflow Spark Declarative Pipelines（原 DLT）
- `ddl-to-data-dictionary` — DDL 翻数据字典

### 选型结论：本地 CLI + Skills

四个理由：

1. **官方维护** — 不靠第三方，能力全集
2. **Azure 中国 endpoint 原生适配** — 不用等社区实现
3. **加新能力直接调子命令** — 不改 Server 源码
4. **调试链路短** — 报错直接可见，不经过 MCP 协议层

> 远程 MCP Server 不是不能用，而是「能用多少看 Server 暴露多少」——这个不确定性对工作流是硬伤。

## P11 · 2.2 测试工作流（Jobs 模式）

### 9 步闭环 · 不动现有 jobs 编排

```
1. export 云上 .py    →   2. Claude 改   →   3. import 回云
                                                       ↓
6. export-run    ←   5. run-now    ←   4. 建临时 job
       ↓
7. 解码 base64  →   8. Claude 分析   →   9. 回到 2 ↻
```

详细步骤：

1. **用户 export 云上 .py** — 从 Databricks workspace 导出 notebook
2. **Claude 改本地 .py** — 在 workspace 编辑，迭代逻辑
3. **用户 import 回云** — 把改好的 .py 导回 workspace
4. **建临时 job** — 不动现有 jobs 编排，新建一个临时 job 跑测试
5. **Claude `jobs run-now`** — 触发 job
6. **Claude `jobs export-run`** — 抓取执行结果
7. **Claude 解码 base64+URL** — export-run 输出是 base64+URL 编码
8. **Claude 自主分析** — 看 stdout/stderr、数据样本、schema
9. **回到步骤 2** — 根据结果迭代，循环往复

### 踩坑点

- **`get-run-output` 只收 `dbutils.notebook.exit()`** — 想拿完整 cell 输出要用 `export-run`
- **Azure 中国禁 `/api/1.2/commands` 端点** — 不能用旧版 commands API
- **输出是 base64+URL 双重编码** — 解码两层才能拿到可读文本

### 规则

- 不动现有 jobs 编排
- 不跑 workspace import/export（用户手动操作）
- 用 all-purpose cluster（不用 SQL Warehouse）
- 临时 job 跑完即弃

## P12 · 2.3 实战示例（2026-07-29）

### test job 配置

- **job_id**: `10761020612585`
- **notebook**: `/Shared/powerlink_warehouse/other/test`
- **cluster**: all-purpose
- **2 cells**: Python + SQL

### 执行结果 · 全链路通

- ✅ 18 秒跑完，无报错
- ✅ Python cell 设参数成功
- ✅ SQL cell 返回数据
- ✅ export-run 拿到全 cell
- ✅ base64+URL 解码成可读文本
- ✅ stdout / stderr 分离提取

### 自主抓取数据

**Cell 0 (Python)** — 设参数：

```python
spark.conf.set("param.bizdate", "20260707")
```

**Cell 1 (SQL)** — 查数据：

```sql
select * from powerlink_prod.pw_ads.
  ads_pw_credit_metric_df
where dt='${param.bizdate}'
limit 10
```

**结果**：10 行 × 68 列完整数据，含 `sap_code` / `company_name` / `company_paydex` 等。

### Claude 自主迭代

拿到数据后 Claude 不只是看一眼就完，会自主推断：

- `dt` 是分区列 → 自动加 `where` 条件避免全表扫描
- `sap_code` → 推断 SAP 主数据关联
- 10 行样本看 schema，不拉全量（省 token、省时间）

这个「样本看 schema，不拉全量」的能力是 9 步闭环的价值——Claude 不需要人告诉它怎么看数据，它自己会推断。

---

下一篇：[Claude Code 工作流（三）：内置功能 — agents view + /insights](/posts/claude-code-workflow-3-cc-features/) — 一屏管理所有后台会话的状态体系，历史会话分析报告的正确用法与常见误区。

> 回到本专栏首页：[Claude Code 使用分享](/topic/claude-code-tips/)。
