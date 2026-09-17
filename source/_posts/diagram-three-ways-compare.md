---
title: AI流程图制作
date: 2026-09-01 14:00:00
tags: [Claude Code, drawio, archify, 流程图, 交付文档]
excerpt: AI 制流程图试了三条路线：Claude 直接写 .drawio XML、跑四个脚本管线导出 PNG；需求交给 archify skill，生成带明暗双主题的可交互单文件 HTML；workbuddy skillhub 一键生成 .cw（d2 语法）源文件 + HTML + PNG，再用 Claude Code 修改细节。同一张 DWS 汇总加工 6 步图当样张，对比三种方式的制作、风格和最终效果，最后说说交付为什么选了 drawio。
---

PowerLink 数仓交付文档需要 10 张流程图。08-26 到 08-31 这一周我先后试了三种 AI 辅助制图方式，最后定稿 drawio 为唯一交付格式，另外两套归档留档。这篇文章拿同一张图（03_dws_steps，DWS 汇总加工与币种转换核心 6 步图）把三种方式摆在一起，比制作方式、风格和最终效果。

样张内容先交代一下：两张源表（`dwd_external_metric_df` 三方指标 / `dwd_internal_metric_df` 内部指标）汇入 6 步加工链，76 列 LEFT JOIN 融合 → 币种识别 → 汇率关联 `dim_fixed_exchange_rate` → 本币金额计算 → `company_paydex` 兜底 → 产出 `dws_pw_credit_metric_df`。图不复杂，但正好覆盖制图时最费手的几个要素：双源 fan-in、串行链、层级语义。

## 方式一：drawio MCP + 脚本管线（最终交付选择）

### 制作方式

Claude 直接写 `.drawio` 的 XML 源码（`mxGraphModel` 骨架：标题 / 节点 / 边 / 图例），drawio MCP 工具负责在编辑器里打开预览和校验。写完不直接交付，按顺序跑一条四个幂等 Python 脚本的管线，再用 draw.io 桌面版 CLI 导出：

1. `_rebrand_drawio.py`：注入品牌框架，上下三色线（红 66% + 白缝 2% + 蓝 32%）、左上蓝色标题、右上 Power/Link 徽章、底部单条图例带。
2. `_add_margins.py`：四边各留 30px（3 网格）外边距。
3. `_normalize_gaps.py`：标题底到首行内容、末行内容到图例顶统一 40px 间距。
4. `_inject_grid.py`：把网格烘焙成绝对坐标的 edge 线段垫底，draw.io CLI 导出没有「带网格」的开关，只能自己画。

最后 `/Applications/draw.io.app/.../draw.io --export --format png --scale 2` 导出，对着 8 项验收清单逐张目检。

这条路上的坑记了几笔：CLI 导出失败也返回 exit 0，必须验证 PNG 存在且目检；cell id 里不能出现 `join` 子串，导出会坏。

规则沉淀是这套方式最值钱的部分。节点按语义配色（绿加工、橙邮件告警、蓝 ODS、金 DIM、灰临时、红 cron），连线颜色等于上游源节点所属数仓层，同层多源用 junction 技法收成单箭头，正交走廊路由禁止穿节点，标签线上方/右侧不压线，节点少的图用左右流向 S 型换行。这些全部写进了一份《drawio 图制作规范与通用 Prompt 模板》，下一张图复制模板填空即可。

### 风格与效果

![drawio 版 03_dws_steps：左右 S 型流向 + 品牌框架 + 网格底](https://img.nightfall7.top/pic/blog_material/drawio.png)

企业交付文档风：网格纸底 + 品牌框架 + 语义配色。这张图节点少，按规范走了左右流向 S 型换行，双源 fan-in 在左，6 步分两行右行、折返左行，整图高度比竖版减半，交付文档里一屏看全。线色一眼读出上游层（绿线来自 DWD、金线来自 DIM），fan-in 双源汇成单箭头进 LEFT JOIN，图面干净。

## 方式二：archify skill（产品感最强）

### 制作方式

把需求交给 archify skill，产出的不是图片而是一个单文件 HTML。机制是 Agent 先写一份 typed JSON 中间表示，archify 自带的 Node CLI 再把它确定性编译成 inline SVG。和其他两种方式同一条路子：AI 写源码、程序出图，只是源码换成了 JSON。用的版本是 v2.16.0，2026-08-30 发布，比我做对比早一天；MIT 许可，支持 architecture、workflow、sequence、dataflow、lifecycle 五种图，数仓流程图落在后两类。图本身可交互：节点搜索、上下游追溯、路径探测、明暗主题切换、PNG/SVG/WebP/WebM 导出，都在右上工具栏里。

这条管线卖的是门禁，不是画图本身，一共三步。先 `validate`，showcase 档 9 项检查全过、0 错 0 警；再 `deliver`，通过了才原子替换 HTML，输出 SHA-256 和字节数，哪一项没过就保留上一版不动；最后 `visual-check`，在 1440×900 到 2048×1320 四档分辨率量首屏包含性，明暗两主题取两端尺寸截图，校验项记在 `visual-check.json`。三种证据各管各的：deliver 证明产物确定，visual-check 证明真实浏览器里行为有界，好不好看还是要人眼审。校验失败时诊断信息会写明错在哪（subject）、量到什么（evidence）、允许哪几种修法（supportedFixes），修两轮错误数还不降就停下如实报告。单文件零外部依赖，离线也能打开。

### 风格与效果

![archify 版 light 主题：虚线 zone 分层 + 等宽字体表名 + 底部证据卡片](https://img.nightfall7.top/pic/blog_material/archify-light.png)

![archify 版 dark 主题：深紫/墨绿全套配色，不是简单反色](https://img.nightfall7.top/pic/blog_material/archify-dark.png)

Web 产品风：虚线 zone 按层分区（01 接入层 / 02 加工层 / 03 产出层），表名等宽字体，步骤节点带编号徽标，左下图例。特别的是底部三张信息卡片：证据来源、核心 JOIN 条件、稳定性建议。「这张图凭什么这么画」的溯源信息直接做进交付物，另外两种方式都没有。dark 主题是单独的深紫/墨绿配色方案，不是简单反色。

### 差异点

它交付的是「应用」不是「图」。演示、汇报场景体验最好，双击 HTML 就能开。作者的定位也说得直白：把技术意图编译成沟通产物，不做通用画图编辑器，布局由 Agent 判断，skill 负责门禁校验和渲染，没有 WYSIWYG 画布。这也解释了它为什么把力气花在产物的可信度和交互上。但单文件约 700KB，进交付文档还是得截成 PNG；整体气质更接近产品白皮书插图，和数仓交付文档的「工程图纸」语境不太贴。

## 方式三：workbuddy skillhub 一键生成 + Claude Code 修改（起步最快）

### 制作方式

workbuddy 的「架构图一键生成」产出三件套：`.cw` 源文件 + HTML 预览 + PNG。`.cw` 本质是 d2 语法：节点写 `shape/class/style`，边写 `源 -> 目标: 标签`，顶部一块 zone classes 样式表，布局完全交给 d2 的 ELK 引擎。再生成管线：`d2 --layout elk` 出 SVG → splice 进 HTML → `_inject_brand.py` 注入品牌框架 → playwright 2x 截图。预览 HTML 依赖 d2 的 CDN 脚本，离线打不开，archify 的单文件没这个问题。

v1 是一键生成的开箱效果；之后交给 Claude Code 修改细节产出 v2，落实 7 条排版规则：居中、大模块级连线、不挡文字、小模块同尺寸、行宽一致、子模块对齐、网格背景。修改过程中踩了 d2/ELK 的坑：`direction: right` 在子节点存在真实边时会被忽略；网格背景要透出必须同时改掉背景 rect 的 `fill` 属性和内嵌 CSS `.fill-N7`，CSS 优先级高于属性，只改一个无效；节点宽高是精确值，label 比 width 宽会溢出框外。

### 风格与效果

![workbuddy v2：竖直单链 + 网格底 + 蓝色 signal 边框](https://img.nightfall7.top/pic/blog_material/workbuddy-v2.png)

简洁竖直单链风：白底圆角节点带投影，signal 类节点（汇率关联、兜底）蓝框强调，边标签灰字骑线居中，自上而下 ELK 自动布局。品牌注入后三色线、图例带与 drawio 同一套视觉词汇。图里是加了网格纸底的 v2，v1 没有网格，排版也松一些。

### 差异点

起步最快，一句话出初稿，`.cw` 源码最简洁、最好维护。但布局全委托 ELK，风格控制力最弱，想让它遵守企业规范（线色分层、junction 合箭、走廊路由）基本要逐条和布局引擎搏斗。竖直长链的信息密度也不如左右流向。

## 横向对比

| 维度 | drawio MCP + 脚本管线 | archify skill | workbuddy 一键生成 + Claude 修改 |
|---|---|---|---|
| 产出物 | `.drawio` 源 + PNG | 单文件 HTML + 校验截图 | `.cw`(d2) + HTML + PNG |
| 布局谁做主 | 自己（绝对坐标 XML） | Agent 定布局，skill 把门禁 | d2 ELK 引擎 |
| 风格控制力 | 最细，规则可沉淀成 Prompt 模板 | 中，主题/预设级 | 最弱，与布局引擎搏斗 |
| 交互能力 | 无（静态图，源文件可用 draw.io 打开改） | 最强：双主题 + 多格式导出 + 可浏览 | 预览 HTML |
| 语义表达 | 线色=上游层、junction 合箭、语义配色 | zone 分层 + 证据卡片 | class 级配色（entity/signal） |
| 信息密度 | 高（左右 S 型流向减半高度） | 中 | 低（竖直长链） |
| 交付文档贴合 | 最贴 | 偏产品白皮书 | 中等 |
| 适合场景 | 正式交付、要长期维护的图 | 演示、汇报、要「好看」的场合 | 快速出草稿 |

## 选择

08-31 对比定稿：drawio 为唯一交付格式，10 张图全部定稿在 `file/diagrams/`；workbuddy 两版归档到 `file/diagrams_bak/`，archify 产出留在 `file/diagrams_archify/` 做演示备用。

三种方式都是「AI 写源码 + 脚本/CLI 出图」，差别在布局控制权在谁手里。drawio 的绝对坐标 XML 控制最细，门槛和管线也最长；archify 把布局判断交给 Agent，skill 专做门禁校验和渲染，交付物包装得最好；workbuddy 把布局交给 ELK，起步最快但最难贴合规范。我的体会是，图越是要进正式交付文档、越是要长期维护，越值得选那个「规则能沉淀成 Prompt 模板、下次复制即用」的方式。drawio 胜出就是这个原因。
