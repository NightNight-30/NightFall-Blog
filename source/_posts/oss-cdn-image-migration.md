---
title: 博客图片迁移到 OSS + CDN
date: 2026-09-17 14:00:00
tags: [阿里云, OSS, CDN, 博客优化, 图片托管]
topic: hexo-tutorial
excerpt: 把博客图片从 GitHub Pages 搬到阿里云 OSS + CDN，记录从建桶、开 CDN、配 HTTPS 证书到替换引用的完整流程。
---

## 为什么要搬

博客之前图片全放在 `hexo_wsj/source/img/` 里，跟着源码走 GitHub Pages 的 CDN。能用，但有两个问题。

GitHub Pages 的 CDN 节点主要在海外，国内读者加载图片要绕一大圈。几张 2-3MB 的背景图，首屏加载慢半拍。另外图片跟着源码进 git 历史，仓库越来越大，clone 和 push 都变慢。

OSS + CDN 的方案很简单：图片存阿里云对象存储，前端通过 CDN 域名访问。CDN 国内节点多，缓存命中后不回源，加载快、费用低。GitHub Pages 那边只留 favicon、备案图标这种几 KB 的小文件。

11 张图，从建桶到上线花了半天。按步骤拆开写。

---

## 一、创建 OSS 桶

OSS 控制台新建 Bucket：

| 配置项 | 值 | 说明 |
|---|---|---|
| Bucket 名称 | `pic-nightfall` | 和博客名对应，一眼认出 |
| 地域 | 华东2（上海） | 读者主要在国内，离用户近 |
| 存储类型 | 标准存储（同城冗余） | 图片读多写少，标准存储最合适 |
| 读写权限 | 公共读 | CDN 回源要能直接读到文件，私有权限每次访问都要签名，没必要 |

建好后进「权限控制」标签页确认 ACL 是「公共读」。这一步我一开始没注意，后面 CDN 回源报 403，排查了半天才发现是权限问题。

---

## 二、添加 CDN 加速域名

### 2.1 CDN 控制台添加域名

进 [CDN 控制台](https://cdn.console.aliyun.com/) → 域名管理 → 添加域名：

| 配置项 | 值 |
|---|---|
| 加速域名 | `img.nightfall7.top` |
| 加速区域 | 仅中国内地（个人博客不需要全球加速） |
| 业务类型 | 图片小文件 |
| 源站类型 | **OSS 域名**（不要选 IP） |
| 源站地址 | `pic-nightfall.oss-cn-shanghai.aliyuncs.com` |
| 回源端口 | 443 |

源站类型选 OSS 域名而不是 IP，CDN 走内网回源 OSS，内网流量免费且延迟更低。选 IP 类型走公网回源，多花钱还慢。

### 2.2 域名归属验证

首次添加域名，阿里云要求验证域名归属权。选 DNS 验证，它会给一个 TXT 记录：

- 主机记录：`verification`
- 记录值：`verify_xxx`（一串哈希值）

去 DNS 控制台（`nightfall7.top`）加这条 TXT 记录，等一两分钟生效后回 CDN 点「点击验证」。

### 2.3 缓存配置

缓存过期时间根目录 `/` 设 1 个月。个人博客图片几乎不变，长缓存命中率高。

忽略参数那一项，模式选「保留指定参数」（留空列表），**保留回源参数选否**。博客 URL 里经常带 `?v=20260917` 这种版本号做缓存刷新，忽略参数后不同版本的 URL 命中同一条缓存，回源时也不该把 `?v=` 带到 OSS，白跑一趟。

### 2.4 关掉费用封顶

这一步容易踩坑。CDN 默认开了三项封顶：流量封顶（50GB/小时）、带宽封顶（10Gbps）、HTTPS 请求数封顶（100 万次/小时）。前两项对个人博客毫无意义。

关键是 HTTPS 请求数封顶。100 万次/小时看起来不小，但博客一个页面加载带十几张图片请求，如果被爬虫扫或者某篇文章被转发，轻松触发。触发后域名直接下线 1 个月。

三项都删掉。个人博客按量计费每月就几块钱，真出问题了云监控会告警，手动处理就行。

### 2.5 DNS 加 CNAME

CDN 添加成功后会给一个 CNAME 值（类似 `img.nightfall7.top.w.kunlunaq.com`）。去 DNS 控制台给 `img.nightfall7.top` 加 CNAME：

| 字段 | 值 |
|---|---|
| 记录类型 | `CNAME` |
| 主机记录 | `img` |
| 记录值 | CDN 给的 CNAME 地址 |

用 `nslookup img.nightfall7.top` 验证解析是否生效。

---

## 三、配 HTTPS 证书

主站 `nightfall7.top` 已经是 HTTPS，图片走 HTTP 加载的话浏览器会报 mixed content 警告，甚至直接阻断。HTTPS 必开。

### 3.1 申请免费证书

进 [SSL 证书控制台](https://yundunnext.console.aliyun.com/?p=cas) → 个人测试证书 → 去购买 → 选「个人测试证书（免费）」+「基础版」。0 元，下单后填写申请：

- 绑定域名：`img.nightfall7.top`
- 验证方式：DNS 自动验证（阿里云 DNS 会自动加 `_dnsauth` 的 TXT 记录，秒过）

免费 DV 证书有效期 90 天，不支持自动续期，到期前手动重新申请一张。

### 3.2 CDN 开启 HTTPS

回 CDN 控制台 → `img.nightfall7.top` → HTTPS 配置：

1. 开启 HTTPS 安全加速
2. 证书选「云盾（SSL 证书服务）」，下拉框选刚签发的证书
3. 开启 HTTP → HTTPS 强制跳转

配好后访问 `http://img.nightfall7.top/xxx` 会返回 301 跳到 `https://...`。

---

## 四、上传图片到 OSS

桶里建目录 `pic/blog_material/`，把图片传进去。ossutil 命令行批量上传或者控制台手动拖拽都行。

OSS 的路径就是 URL 的路径。`pic/blog_material/bg-mountains.jpg` 对应访问地址 `https://img.nightfall7.top/pic/blog_material/bg-mountains.jpg`。

传完后用 curl 验证一下：
```bash
curl -sI https://img.nightfall7.top/pic/blog_material/bg-mountains.jpg
# 应返回 200 OK + Content-Type: image/jpeg
```

---

## 五、替换博客里的图片引用

先 grep 找出所有 `/img/` 路径的图片引用。Markdown 文件里有 cover 字段、banner 的 `bg:` 参数、文章内 `![](/img/...)` 引用，CSS 文件里有 `url(/img/...)` 背景图引用。

把 `/img/xxx.jpg` 改成 `https://img.nightfall7.top/pic/blog_material/xxx.jpg`，涉及的文件：

| 文件 | 引用位置 |
|---|---|
| `_posts/nightfall-blog-build.md` | cover + banner 背景图 |
| `_posts/diagram-three-ways-compare.md` | 4 张流程图对比图 |
| `about/index.md` | banner 背景图 |
| `friends/index.md` | banner 背景图 |
| `life/index.md` | banner + swiper 相册 |
| `shuoshuo/index.md` | banner 背景图 |
| `css/code-enhance.css` | 全站背景图（light/dark 3 处） |

替换完跑 `hexo clean && hexo g`，检查生成的 HTML 里都是 CDN 地址。

图片已经迁到 OSS，GitHub Pages 那边的原文件没必要留着了。从 `source/img/` 移走，`hexo g` 后就不会出现在 public 目录里。本地留一份备份放博客根目录的 `backup_img_cdn_migration/`，然后 `hexo d` 部署，GitHub Pages 仓库里这些图片就删掉了。

---

## 六、配 Referer 防盗链

Bucket 名和 OSS 域名是公开信息，知道的人可以写脚本刷流量消耗你的钱。CDN 层面配防盗链能拦截非博客页面的请求。

CDN 控制台 → `img.nightfall7.top` → 访问控制 → 防盗链 → 修改配置：

| 配置项 | 值 |
|---|---|
| 防盗链类型 | Referer 白名单 |
| 规则（回车分隔） | `*.nightfall7.top` + `nightfall7.top` |
| 允许通过浏览器地址栏直接访问资源 URL | **不勾选** |
| 忽略 scheme | **勾选**（HTTP/HTTPS 都匹配） |
| 重定向 URL | 留空（拦截返回 403） |

配完后：
```bash
# 博客页面正常加载（带 Referer）→ 200
curl -sI -H "Referer: https://nightfall7.top/" https://img.nightfall7.top/pic/blog_material/bg-mountains.jpg

# 外部盗链 → 403
curl -sI -H "Referer: https://other-site.com/" https://img.nightfall7.top/pic/blog_material/bg-mountains.jpg

# 空 Referer（直接访问）→ 403
curl -sI https://img.nightfall7.top/pic/blog_material/bg-mountains.jpg
```

---

## 费用

上线后的开销：

| 项目 | 费用 |
|---|---|
| OSS 存储（约 15MB） | 几乎为 0 |
| OSS 外网流出（CDN 回源流量） | CDN 缓存命中后不回源，极少 |
| CDN 流量 | 按量计费，每月几块钱 |
| CDN HTTPS 请求数 | 比 HTTP 多约 0.05 元/万次 |
| SSL 证书 | 0 元（免费 DV 证书，90 天有效期） |

总费用每月大概 5-10 块，换来的是国内加载速度明显提升。

## 后续要做的

免费证书 90 天到期，到期前手动重新申请一张，CDN 控制台替换就行。防盗链已经配好，外部刷流量和盗链都被拦截了。

`hexo_wsj/source/img/` 里现在只留了 favicon、备案图标等 8 个小文件，都是 footer 里还在用的。大尺寸背景图和流程图全部走 CDN。
