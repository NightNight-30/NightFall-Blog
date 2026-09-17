---
title: 博客搭建记录（五）：部署、域名与备案
date: 2026-09-04 16:30:00
tags: [hexo, stellar, 博客搭建]
topic: hexo-tutorial
excerpt: 静态站放 GitHub Pages，后端服务走阿里云。中间隔着 HTTPS、域名和 ICP 备案，一步一步蹚过来。
---

最终的架构是这样的：博客静态站放 GitHub Pages，自定义域名 CNAME 过去，HTTPS 由 GitHub 代管；一台阿里云轻量服务器跑说说和评论两个容器，用子域名 `api.nightfall7.top` 加 nginx 反代对外。

为什么非要 HTTPS：博客前端在 GitHub Pages 上强制 HTTPS，前端调 HTTP 的后端接口会被浏览器混合内容策略拦截。后端要有合法证书，Let's Encrypt 不给裸 IP 签证书，于是必须有域名。整个域名探索就是这么开始的。

## GitHub Pages 部署

6 月 30 日第一次部署，步骤不多：

1. 建仓库 `<用户名>.github.io`，Public、空仓库。用户名加 `.github.io` 这个格式会自动启用 Pages。
2. 装 hexo-deployer-git，`_config.yml` 里 `url` 改成仓库地址，deploy 配成 git 类型。
3. `hexo clean && hexo generate`，然后 `hexo deploy`。
4. 等构建：`gh api repos/<用户名>/<用户名>.github.io/pages`，status 从 building 变 built，约 45 秒。

源码仓和 Pages 仓分开：源码仓可以分支开发，Pages 仓是纯公开产物。

装部署插件时碰到 npm 报 EACCES：之前 sudo 装包留下了 root 属主的缓存文件。临时绕过是换个缓存目录，根治是 `chown` 修回 `~/.npm` 的属主。

## canonical 的盗站误报

部署当天，首页顶部弹出「本站为非法克隆站」的警告——配置明明是对的。

查 `window.canonical`，发现 `originalHost` 写的是大写开头的域名，而 `location.host` 是全小写。浏览器对 URL 的 host 自动做小写规范化，精确匹配失败，主题把主站当成了盗站。配置里沿用大写习惯的地方全部改小写。

改完重新部署还是弹：GitHub Pages 的 CDN 缓存了旧版本，清浏览器缓存再验证才通过。

## 域名与备案

服务器在大陆，80 和 443 端口要 ICP 备案才能用。这段走了不少弯路。

先试免费的 DuckDNS 子域名，DNS 解析正常，但签证书时 80 端口被重置。curl 过去返回 403，Server 头是陌生的，不是 nginx 回的。查下来是云厂商的代理在基础设施层拦截 80/443，未备案域名直接 403，流量根本到不了实例内的 nginx，实例里的防火墙和端口监听都看不到这一层。

DuckDNS 的三级子域名也没法在控制台绑定备案。于是买了个 `.top` 域名。国内注册商才能备案，`.top` 首年便宜。

备案要 1 到 3 周，期间用 8443 非标端口过渡：非标端口不查备案，证书用 DNS-01 方式签（80 不通，HTTP-01 用不了）。备案审核期间 80/443 不能跑网站，会显示未备案提示页，还可能导致备案被拒。

这段时间沉淀了几条经验：

- 域名实名认证和 ICP 备案是两套流程，前者 1 到 3 天，后者 1 到 3 周。
- 个人备案的网站名称有雷区：不能带国家、政府字样，不能太商业化，论坛、社区、新闻这类需要专项审批的也不行。
- 备案期间非标端口是可用的临时通道。

手机端还有个插曲：华为手机访问 8443 全部连接被重置，四个网络组合排查都复现，结论是非标端口被运营商深度包检测拦了。这个问题留到切 443 后自然消失。

## 备案通过，切回 443

7 月 10 日备案下来，一次切换动了五处，严格按顺序执行：

1. **DNS**：删掉指向服务器的 A 记录，加根域 CNAME 指向 GitHub Pages（阿里云支持根域 CNAME 扁平化），另加一条 `api` 的 A 记录指向服务器。
2. **GitHub Pages**：后台填自定义域名，会自动创建 CNAME 文件；勾强制 HTTPS，Let's Encrypt 自动签发。
3. **防火墙**：放行 80 和 443。80 要开着，证书续期走 HTTP-01。
4. **服务器**：证书从 DNS-01 换成 `certbot --nginx` 自动签发，以后自动续期；nginx 配置改监听 443；Artalk 的 CORS 白名单追加新域名并重启容器。
5. **清理**：删 8443 防火墙规则、删旧证书、删白名单里的 8443 条目。

切换前本地配置先全部改好提交，但不部署。DNS、Pages、服务器没就绪时部署，盗站警告和接口失败会同时出现。

部署后跑了一份 15 项的验证清单：canonical 无误报、页脚备案号显示、两个域名 dig 解析、证书链校验、说说和评论接口 200、CORS 预检 204。全过。

有一处灰色地带，如实说明：域名 CNAME 到 GitHub Pages 而不是备案登记的服务器，备案在技术上是悬空的。实际操作里只要页脚挂备案号、网站可访问即可，这也是多数独立博客的做法。

## 部署失败的恢复模式

hexo deploy 偶发 `SSL_ERROR_SYSCALL`：commit 已经生成，push 失败。恢复方式是进 `.deploy_git` 目录手动补推：

```bash
cd .deploy_git
git remote add origin https://github.com/<用户名>/<用户名>.github.io.git
git push origin master:main
```

坑在于 `.deploy_git` 的 remote 不持久，每次 `hexo clean` 重建目录就没了，下次失败要重新加。这是 hexo-deployer-git 的行为，不是 bug。

## 收尾

现在域名的分工很清晰：根域走 GitHub Pages 的 CDN，api 子域走自己的服务器。证书两边都是自动续期。

备案是大陆个人博客绕不开的一环，整个过程最值钱的结论是：备案期间用非标端口过渡，备案通过后再按顺序切换，别跳步。
