---
title: 博客搭建记录（三）：说说与评论系统部署
date: 2026-09-04 16:10:00
tags: [hexo, stellar, 博客搭建]
topic: hexo-tutorial
excerpt: Ech0 说说和 Artalk 评论，两个 Docker 容器，从本机跑到云服务器。瀑布流改了三版，CORS 查了一下午。
---

博客有两个功能需要后端：说说（Ech0，自托管微博客）和评论（Artalk）。都是 Docker 容器，先在本机跑通，6 月 29 日迁上云服务器，之后一直只靠 nginx 反代对外。

这篇记录两个服务的部署、页面集成，以及迁移和跨域踩的坑。

## Ech0 本地部署

参考别人家的说说页起步，目标效果是两列瀑布流加图片灯箱。本机一条命令起容器：

```bash
docker run -d --name ech0 -p 6277:6277 \
  -v $HOME/ech0/data:/app/data \
  -e JWT_SECRET=$(openssl rand -hex 32) \
  sn0wl1n/ech0:latest
```

JWT_SECRET 用随机值生成，存一份到数据目录，后面迁移要跟着走。

接口文档不用猜，容器自带 Swagger（`/swagger/index.html`，90 个路径）。前端要用的就一个：`POST /api/echo/query`，body 传 `{page, pageSize}`，返回说说列表。实测有两个坑点：

- 图片文件的 url 是相对路径，前端要自己拼服务地址前缀；
- 头像没有公开接口，`/api/files/avatar/*` 全 404，只能在前端配置里硬编码头像地址。参考的那个站点也是这么干的。

## 瀑布流改了三版

v1 单列卡片，先跑通数据。v2 用 CSS `column-count: 2`，两列是出来了，但它是先填满左列再填右列，阅读顺序断裂。v3 换 JS 布局：卡片绝对定位，每张新卡放到当前高度较小的那一列，窗口缩放和图片加载完重排。

抄参考站源码时顺带加了 `transform: scale(0.9)`，后来吃了苦头：缩放不影响 `offsetHeight` 的返回值，布局计算在未缩放的坐标系里做，视觉缩放在上层叠加，容器怎么都对不齐。最后直接删掉这个缩放。

卡片样式初版是紫粉渐变加毛玻璃，8 月的整体改版里换成了扁平卡片，视觉演进另有一篇：{% post_link visual-style-tuning %}。

## 互动三件套

说说卡片下面一排点赞、评论、分享，三个独立的 JS 文件，各自干一件事：

- **点赞**：`PUT /api/echo/like/{id}`。localStorage 记已赞列表防重复，点击乐观更新，请求失败回滚。后端做了幂等，同一来源一小时内重复点赞不累加。后来改成可取消的 toggle。服务端没有 unlike 接口，取消只在前端生效，计数保留累计值。
- **评论**：用的是 Ech0 自带的评论系统，支持嵌套回复和待审状态。展开评论框先取 form_token，配一个蜜罐隐藏字段反垃圾。
- **分享**：弹层里放二维码和复制链接。二维码最初想用 npm 的 qrcode 包，jsdelivr 加载成功但没暴露全局变量，CommonJS 模块不能直接 `<script>` 用。换成远程二维码 API，一个 `<img>` 解决，零 JS 依赖。

三个模块都挂在瀑布流上，评论展开收起会改变卡片高度，每次 DOM 变化都要触发一次重排，外加 200ms 延迟兜底。

## Artalk 评论部署

Artalk 也是 Docker 起：

```bash
docker run -d --name artalk -p 23366:23366 \
  -v $HOME/artalk/data:/data --restart unless-stopped \
  artalk/artalk-go
```

建管理员用 `docker exec artalk artalk admin --name ... --password ...`。注意 CLI 要传 flag，不能 pipe 标准输入，它读 TTY，pipe 会报 ioctl 错误。

Stellar 原生支持 artalk，配置四行。踩坑集中在细节上：

1. `site:` 和后台站点名逐字一致，大小写敏感，不一致评论就加载不出来。
2. `server:` 末尾不带斜杠。
3. 前端 JS/CSS 走 jsdelivr，比 unpkg 稳。
4. 管理后台在 `/sidebar/#/login`，不是 `/ui/admin/`。
5. `darkMode: auto` 先用默认值，主题源码注释里写了其它模式都有问题。

## 迁上云服务器

6 月 29 日迁到阿里云轻量服务器。服务器只跑这两个容器加 nginx，不托管静态站。三个坑：

**Docker 安装走镜像源。** `get.docker.com` 在大陆握手超时，要配阿里云的 yum 源，安装时带 `--disableexcludes=all`，否则被默认排除规则过滤掉。装完配镜像加速器，但加速器只对热门镜像有效，ech0 这种小众镜像不走加速。

**镜像架构要对上。** 本机是 Apple Silicon（ARM64），服务器是 x86_64。本地 `docker save` 出来的镜像放上去就是 `exec format error`。解法：服务器上通过多架构代理直接拉，Docker 自动选 amd64，拉完用 `docker image inspect` 确认架构。

**数据打包迁移。** 数据目录 tar 打包上传解压，两个收尾动作：删掉 macOS 打包带进来的 `._*` 资源叉文件；`chown -R root:root` 修正属主。tar 保留了本机的 UID 501，在 Linux 上映射成一个不相干的用户。JWT_SECRET 一起迁，迁完登录状态还在。

迁完容器端口都绑 `127.0.0.1`，公网只能经 nginx 进来。

## CORS 查了一下午

迁完第二天，前端评论加载失败，`Failed to fetch`。排查链：

1. `fetch` 换 `no-cors` 模式，返回 opaque，说明是 CORS 不是网络不通；
2. curl 带 Origin 请求，响应只有 `Vary: Origin`，没有 `Access-Control-Allow-Origin`；
3. 进容器找配置文件，`/app/data` 下没有——`docker inspect` 看挂载，容器内路径是 `/data`，不是想当然的 `/app/data`；
4. 改 `trusted_domains`，重启容器，ACAO 头出现。

Artalk 的 CORS 白名单就是 `trusted_domains`，三条规则：

- 域名必须全小写。浏览器会把 Origin 的 host 自动转小写，白名单是精确匹配，我配置里写了大写 N 的域名，匹配失败。
- 带端口的 Origin，端口要写全。备案期间用的是 8443 端口版，`https://域名:8443` 整条进白名单。
- Artalk 不热重载配置，改完必须重启容器。

`Vary: Origin` 出现不等于 CORS 通过，它只表示响应随 Origin 变化，要看的是 ACAO 头。

## nginx 补一条反代

8 月发现 Artalk 管理后台登录不了。根因：后台是个 SPA，JS 里 API 路径写死成 `/api/v2`，不带反代前缀。博客前端走的是 `/artalk/api/v2/*`，一直正常；只有后台自己用裸的 `/api/v2`，nginx 没配这段，落到了 ech0 的兜底路由。

补一个 `location /api/v2/` 反代到 Artalk，注意 `proxy_pass` 不带尾斜杠，保留前缀原样转发。`/api/v2/` 比 ech0 的 `/api/` 路径更具体，nginx 最长前缀匹配，两边互不影响。

## 邮件通知试过一次，回退了

8 月 19 日想给访客开邮箱管理能力，配了 163 的 SMTP。实测发现 2.9 的邮箱登录是「邮箱+密码」模式，不是想象中的验证码模式；访客改删评论靠的是评论提交时发的编辑链接邮件，这条链路依赖 SMTP 实发，当时没验证通。加上管理员通知邮箱填的是虚构地址，当天整体回退到匿名评论状态。

邮件功能留着以后再做：先把管理员邮箱换成真实邮箱，再验证发信链路。

## 收尾

现在说说和评论都在同一台服务器上，两个容器绑本机端口，nginx 统一出口。评论走 `/artalk/`，说说 API 走 `/ech0/`，后台直接开根路径。

右栏「近期说说」组件拉的是同一个 Ech0 接口，一份数据两个消费端。
