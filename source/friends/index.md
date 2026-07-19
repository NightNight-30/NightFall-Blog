---
title: 友链
date: 2026-07-17 15:00:00
menu_id: friends
---

{% banner 友链 与有趣的灵魂相遇。 bg:/img/site-bg-mountains.jpg %}{% endbanner %}

<!-- 友链说明：灵感来自 https://xaoxuu.com/blog/20250602/ 动态友链方案 -->
<!-- 数据源：https://github.com/NightNight-30/friends Issues + GitHub Actions 生成 -->
{% note color:indigo 本页友链从 [NightNight-30/friends](https://github.com/NightNight-30/friends) 仓库的 Issues 自动抓取，按最新文章时间排序。想交换友链的请[提 Issue 申请](https://github.com/NightNight-30/friends/issues/new?assignees=&labels=%E5%AE%A1%E6%A0%B8%E4%B8%AD&template=friend-link.yml&title=友链申请：)，审核通过后会自动显示。 %}

<!-- posts:true 开启最新文章展示；jsdelivr CDN 国内访问更稳 -->
{% friends posts:true api:https://cdn.jsdelivr.net/gh/NightNight-30/friends@output/v2/data.json %}

<!-- 评论区 artalk 自动注入 -->
