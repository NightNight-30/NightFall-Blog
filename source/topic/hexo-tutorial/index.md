---
title: Hexo 教程
date: 2026-06-28 10:00:00
topic: hexo-tutorial
menu_id: post
description: Hexo 博客搭建与使用相关文章
---

本专栏记录 NightFall 博客从 0 到 1 的搭建过程，含 Hexo 配置、Stellar 主题定制、Ech0/Artalk 集成、视觉打磨与部署上线。

## 本专栏 OKR

{% okr o1 %}
完成 NightFall 博客 Hexo 教程专栏的内容沉淀
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
<!-- okr kr5 percent:60 -->
源码备份工作流 + 持续内容沉淀
- 源码已备份到 GitHub 公开仓 `NightFall-Blog`（与 Pages 仓分仓存放）
- workspace / 源码仓双副本同步策略已建立（rsync 按需触发）
- 待补：每个 KR 对应的详细教程文章陆续发布
{% endokr %}

## 专栏文章

按发布倒序：

1. **{% post_link nightfall-blog-build %}** — 从零到上线，用 Hexo + Stellar 搭建个人博客的完整过程，涵盖技术选型、架构设计、模块实现、视觉打磨与部署上线。（2026-07-24，置顶）

> 待补文章清单（按 OKR 排）：
> - KR1：`_config.yml` 完整配置详解 / `_config.stellar.yml` 主题定制 / menubar + widgets + footer 配置
> - KR2：Ech0 说说系统 Docker 部署 + masonry 瀑布流 / Artalk 评论系统部署 + nginx 反代 + CORS 修复
> - KR3：整页背景图 + navbar 透明化 / 卡片渐变毛玻璃统一 / 暗色模式调优 / 科技感点睛 3 项 / 思源宋体 / 置顶徽章
> - KR4：GitHub Pages 部署 + hexo-deployer-git / ICP 备案 + 8443 切 443 / apex 与 api 子域 split 架构 / canonical 防盗站
> - KR5：源码备份到 GitHub 公开仓 / workspace 与源码仓双副本同步策略
