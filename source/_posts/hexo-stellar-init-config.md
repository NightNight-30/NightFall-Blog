---
title: 博客搭建记录（二）：Hexo + Stellar 初始化配置
date: 2026-09-04 16:00:00
tags: [hexo, stellar, 博客搭建]
topic: hexo-tutorial
excerpt: 6 月 26 日开工，导航、组件、笔记本、标签插件挨个配过来。记录配置过程，也记录踩的坑。
---

博客 2026 年 6 月 26 日开工，Hexo 8.1.1 加 hexo-theme-stellar 主题。主题是 npm 安装的，放在 node_modules 里，不进 themes/ 目录，也不进 git。日常改的只有三处：`_config.yml`、`_config.stellar.yml`、用户资源目录 `source/`。

这篇记录初始化阶段做的事：导航、页脚、侧边栏组件、笔记本、标签插件，还有几个反复踩的坑。

## 先说热重载规则

Stellar 的本地预览不是什么都热重载，第一天就交了学费：

| 改动类型 | 热重载 | 怎么办 |
|---|---|---|
| `source/_posts/*.md`、`source/_data/*.yml` | 是 | 保存刷新即见 |
| `_config.yml` / `_config.stellar.yml` | 否 | 重启 |
| `node_modules/hexo-theme-stellar/**` | 否 | 重启 |
| 新增笔记、改笔记本配置、改 site_tree | 否 | 重启 |

笔记本那行值得单独说：标签树是在 generateBefore 事件里构建的，server 的热重载不会重新触发这个事件，所以改笔记本相关配置必须重启。

本地预览 `npm run server`，监听 4000；停掉用 `lsof -ti:4000 | xargs -r kill -9`。验证就三招：curl 看首页返回 200，浏览器 Console 看有没有报错，截图对比视觉效果。

## menubar 五项与配色

导航五项一次定稿：博客、生活、技术、读书、关于。图标统一用 solar 图标集的 bold-duotone 风格，每项一个颜色：

- 博客 `#E91E63`（Material pink 500）
- 生活 `#FFC107`（amber 500）
- 技术 `#6366F1`（蓝紫）
- 读书 `#1BCDFC`（青）
- 关于 `#6D28D9`（violet 600）

五个颜色分布在色环不同位置，保证扫一眼能分开。后来「读书」换成「说说」，图标从聊天气泡换成气球，配色没动。

menubar 的 id 要和页面 front-matter 里的 `menu_id` 对应，点击后导航项才会高亮。对不上就是点了不亮，配置本身不报错。

## 图标字典：初始化最大的坑

配菜单图标时发现一个反直觉的行为：`icon:` 字段写了一个图标名，页面上原样显示这串文字，不报错。

原因：Stellar 只渲染它 `_data/icons.yml` 字典里列好的 SVG，字典外的名字直接当文字输出。这里面有两层坑：

1. **图标名本身不存在。** Solar 集有 3000 多个图标，名字全靠猜不行。用 `curl https://api.iconify.design/solar/<name>.svg` 探测，404 就是不存在，200 才有。
2. **图标在 Iconify 存在，但主题字典没有。** 我试的第二个图标 curl 返回 200，页面上还是显示文字。主题字典是静态的，只预置了 10 个 solar bold-duotone 图标。

解法是新建 `source/_data/icons.yml`，把 SVG 内容直接塞进去。主题用对象合并的方式处理字典，用户字典覆盖默认字典。

两个细节：Iconify 下载的 SVG 默认 `width="1em" height="1em"`，建议改成 `width="32" height="32"`，和主题字典里其他图标的固有尺寸一致；图标集合名有对照关系，`fab:git-alt` 对应 API 路径 `fa-brands/git-alt`。

之后加新图标就是固定流程：curl 验证存在 → 复制 SVG → 改 32x32 → 追加进 icons.yml → 重启。

## 页脚和头像

两个小坑，都是 404：

- footer social 的 icon 字段必须填完整 HTML（`<img src="..." alt="GitHub"/>`），只填图标名不渲染；url 留了个空的 `https://`，四个链接全 404，逐个填实。
- 头像用模板变量渲染出 `src="undefined"`，改成 GitHub 头像直链 `https://github.com/<用户名>.png` 解决。

页脚内容区有个实用机制：`{author.name}`、`{theme.name}`、`{theme.version}` 这类花括号占位符是主题渲染时替换的，不要写死值，主题升级后版本号会跟着变。写法从主题默认 footer 里抄。

sitemap 分组前先 curl 实测路由：`/categories/` 返回 404，`/tags/` 和 `/archives/` 返回 200，所以页脚没放分类链接。404 链接挂在页脚上伤 SEO。

另一个省事的习惯：Stellar 默认配置（主题目录下的 `_config.yml`）已经开好了多数功能，复制代码、图片懒加载都是默认开的。配之前先查默认值，`_config.stellar.yml` 里只覆盖想改的项。

## widgets 与 site_tree

1.26.0 起，侧边栏组件改成按页面类型挂载：每种页面类型（home、post、page、notebooks 等）可以独立指定 leftbar 和 rightbar，组件名按逗号分隔的顺序挂载。

我配了三个组件：

- **welcome**：markdown 组件，写在 `source/_data/widgets.yml`，挂到所有页面类型的 leftbar 首位。注意覆盖语义：用户配置里写了同名组件就是整体覆盖，不是字段合并；想禁用内置组件，同名置空即可。
- **timeline**：近期动态，数据源是 GitHub 仓库的 issues API，挂在首页右栏。
- **related**：同栏文章列表。这个组件容易误解：它不按 tags 匹配，而是按文章 front-matter 的 `topic` 字段（专栏）归组。文章指定了专栏、专栏在 `_data/topic/` 里有定义，组件才渲染。

## 笔记本系统

技术笔记用的是 1.29.0 引入的 notebooks 系统：一个笔记本一个 yml（`source/_data/notebooks/<id>.yml`），笔记 md 用 front-matter `notebook: <id>` 归属。

一个设计上的巧合：menubar「技术」指向 `/tech/`，笔记本的 base_dir 也是 tech，于是一套 URL 两种用途——点导航直接进笔记本列表页。前提是 `source/tech/` 下不放 index.md 占用路由。

标签图标配错位置卡了半天：标签图标要配在 `_config.stellar.yml` 的 `notebook.tagcons` 段，key 区分大小写；配在笔记本自己的 yml 里无效，因为渲染标签树的模板只读主题配置里那一段。图标本身还是得在 icons.yml 字典里有定义，否则又是原样输出文字。

## banner 与标签插件

独立页面的顶部横幅用 `{% raw %}{% banner %}{% endraw %}` 标签，三种用法：页面顶部横幅带导航、个人资料页、文章摘要卡片。三个坑：

1. 标题参数不能含空格，主题按空格分词，需要空格的地方用 `&nbsp;` 代替。
2. 背景图渲染成 `<img class="lazy bg">` 子元素，不是 CSS background，调试时别找错地方。
3. 页面用了手写 banner，主题还会自动生成一个顶部横幅，两个打架。用 `:has()` 父选择器把自动横幅隐藏掉。

列表卡片要显示封面图，front-matter 加 `cover` 和 `poster` 两个字段。`excerpt` 必须手填，不然自动摘要会把 banner 文字吃进去。

## 两个会反复遇到的点

**文内链接别硬编码。** Hexo 默认 permalink 是 `:year/:month/:day/:title/`，markdown 里写死 `/posts/slug/` 必 404。正确写法是主题的 {% raw %}`{% post_link slug %}`{% endraw %} 标签，按 permalink 规则自动生成路径。

**搜索用 local_search。** 不依赖后端，配 `service: local_search`，依赖 hexo-generator-searchdb，索引文件在 `/search.xml`。500 篇以内的个人博客够用。

## 收尾

初始化阶段留下了三处 node_modules 补丁：专栏卡链接指向修正、笔记卡片支持封面图、卡片加 photo 类。这些补丁每次 npm update 都会丢，需要重打，清单记在配置参考里。这是 npm 安装主题的代价，也是后来坚持「定制走 inject、不碰主题源码」的原因之一。

配置键在 1.44.0 升级时有一批删改，迁移过程另有一篇：{% post_link stellar-144-upgrade %}。
