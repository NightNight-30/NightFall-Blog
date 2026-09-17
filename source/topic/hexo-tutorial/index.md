---
title: 博客搭建记录
date: 2026-06-28 10:00:00
topic: hexo-tutorial
menu_id: post
description: NightFall 博客的搭建、调优与维护记录
---

本专栏记录 NightFall 博客从 0 到 1 的搭建过程，以及上线后的升级与维护，含 Hexo 配置、Stellar 主题定制、Ech0/Artalk 集成、视觉打磨与部署上线。

## 本专栏 OKR

{% okr o1 %}
完成 NightFall 博客「博客搭建记录」专栏的内容沉淀
来自复盘：博客已上线运行，文章跟着实际开发节奏补
<!-- okr kr1 percent:100 -->
Hexo + Stellar 主题初始化与基础配置
- 已完成：`_config.yml` / `_config.stellar.yml` / menubar / widgets / footer
<!-- okr kr2 percent:100 -->
Ech0 说说 + Artalk 评论系统集成
- Ech0 Docker 部署 + masonry 两列瀑布流已上线
- Artalk 评论系统已部署（nginx 反代 + CORS 修复）
<!-- okr kr3 percent:100 -->
整页背景图 + 卡片渐变毛玻璃视觉系统
- 背景图已配，navbar 已透明化
- 文章/专栏卡片渐变毛玻璃已统一
- 暗色模式细节调优已完成（科技感点睛 3 项 + 思源宋体 + 置顶徽章）
<!-- okr kr4 percent:100 -->
部署到 GitHub Pages + 自定义域名
- 已部署到 `nightfall7.top`（apex 域 CNAME 到 GitHub Pages，走 CDN + 自带 HTTPS）
- ICP 备案通过，8443 临时端口切 443 标准端口
- `api.nightfall7.top` A 到云服务器跑后端 API + ech0 管理后台
<!-- okr kr5 percent:100 -->
源码备份工作流 + 持续内容沉淀
- 源码已备份到 GitHub 公开仓 `NightFall-Blog`（与 Pages 仓分仓存放）
- workspace / 源码仓双副本同步策略已建立（rsync 按需触发）
- 每个 KR 对应的详细教程文章已发布（见下方专栏文章列表）
{% endokr %}

## 专栏文章

按发布升序：

1. **{% post_link nightfall-blog-build %}** — 从零到上线，用 Hexo + Stellar 搭建个人博客的完整过程，涵盖技术选型、架构设计、模块实现、视觉打磨与部署上线。（2026-07-24，置顶）
2. **{% post_link hexo-stellar-init-config %}** — KR1：导航、组件、笔记本、标签插件，以及图标字典那些反复踩的坑。（2026-09-04）
3. **{% post_link ech0-artalk-deploy %}** — KR2：Ech0 说说与 Artalk 评论，两个 Docker 容器从本机到云服务器，瀑布流三版迭代与 CORS 排查。（2026-09-04）
4. **{% post_link visual-style-tuning %}** — KR3：从代码块阴影到整站改版，视觉改动全部走注入层，记录两个月的样式迭代。（2026-09-04）
5. **{% post_link deploy-domain-icp %}** — KR4：GitHub Pages 部署、域名选择与 ICP 备案，8443 过渡到 443 的完整切换。（2026-09-04）
6. **{% post_link source-backup-sync %}** — KR5：源码仓独立备份，双副本 rsync 同步，以及一次历史分叉的处理。（2026-09-04）
7. **{% post_link stellar-144-upgrade %}** — 从 1.33.1 到 1.44.0，两天十一轮对齐，记录主题升级的变更与踩坑。（2026-09-04）
