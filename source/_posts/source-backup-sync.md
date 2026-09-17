---
title: 博客搭建记录（六）：源码备份与双副本同步
date: 2026-09-04 16:40:00
tags: [hexo, stellar, 博客搭建]
topic: hexo-tutorial
excerpt: hexo deploy 推上去的只是构建产物，源码另需要一个仓库。双副本怎么同步，什么时候同步，踩过一次分叉。
---

`hexo deploy` 备份的是什么？只有构建产物，public/ 目录推到 Pages 仓库。配置文件、文章源稿、自己写的 JS 和 CSS，全都不在里面。硬盘一坏，线上还在，源码没了。

源码得另找一个地方放。

## 两个仓库

| 仓库 | 内容 | 用途 |
|---|---|---|
| NightFall-Blog | 源码：配置、source/、scaffolds、nginx 配置、package.json | 备份与开发 |
| `<用户名>.github.io` | hexo deploy 推的静态文件 | Pages 托管 |

两个仓库都是公开的。源码仓公开的前提是推送前扫一遍敏感信息，后面细说。

## 建仓

6 月 26 日，项目初始化当天就建了源码仓：

```bash
git init -b main
gh repo create NightFall-Blog --public --source . --remote origin --push
```

`-b main` 直接指定默认分支，免得后期改名。gh CLI 一条命令完成建远程仓、关联、首推。当时 Pages 部署还排不上日程，源码备份先于部署存在。

另外克隆了一份干净副本放在别处，没有 node_modules，用来校验推送结果。

## 双副本怎么同步

主编辑目录是工作区，带 node_modules，随时能跑 hexo；副本是干净克隆，只做校验。同步用 rsync，单向，从工作区到副本：

```bash
rsync -a --checksum \
  --exclude="node_modules/" \
  --exclude="public/" \
  --exclude=".deploy_git/" \
  --exclude=".git/" \
  --exclude="db.json" \
  工作区/ 副本/
```

两个参数有讲究：

- `--checksum` 按内容差异判断，不用默认的修改时间。备份场景里 mtime 经常不可靠。
- 必须排除 `.git/`。两个目录的 git 是独立实例，不是 worktree，rsync 带过去就把副本的提交历史覆盖了。

同步完进副本看 diff，按需 add、commit、push。

同步是低频操作，不跟着每次编辑走。日常发布靠 `hexo deploy`，源码同步攒到一批再做：一个功能模块落地、一批视觉调整完成，或者一次发版之后。触发粒度有讲究，后面吃过亏。

`.gitignore` 把构建产物和开发垃圾全挡在外面：node_modules、public、db.json、编辑器交换文件、调试截图。依赖靠 package.json 和 lock 文件重装，截图这类调试产物保持 untracked，源码仓只进可复现构建所需的东西。

## 公开仓推送前扫一遍

源码仓是公开的，推送前固定做敏感信息扫描：密码、SSH key、token、.env，关键词过一遍，零命中。

判断标准分三档：凭据和 token 绝不能进；本就公开可探测的不算泄露，比如服务器 IP，DNS 的 A 记录谁都能查；只在本机可达的反代端口和标准路径，留着也无妨。nginx 配置里有这三类信息，核对后原样推送。

## 攒太久会分叉

7 月 10 日补过一次存量提交：一周的配置改动攒成一个大 commit，39 个文件，事后拆不开。这是「按需触发」的第一次代价，还能接受。

9 月 4 日付出了第二次代价。Stellar 1.44.0 升级做完，128 个文件的改动要进备份仓，新分支推上去开 PR——状态直接是冲突。

查下来：备份仓的远端停在 7 月 27 日，八月的专栏体系和这次升级从未推过；本地 main 在同期还有三笔自己的提交。两边历史分叉了。

核对远端独有的文件：测试文章、旧背景图、已下线的脚本，全是后续有意删掉的遗留，全树 grep 确认零引用，删得安全。于是用 `-s ours` 策略合并：

```bash
git merge -s ours origin/main
git push
gh pr merge --squash
```

`-s ours` 保留本侧的文件树，只把对方的历史连线并进来，正适合「对方有的改动我这边都已刻意删除」的场景。合并前后各做一件事：合并前逐一核对对方独有文件是否真的该丢；合并后 diff 验证文件树没变。

本地的三笔旧提交开了个备份分支留着防丢。

## 收尾

几个月下来，备份这件事的经验就四条：

1. 构建产物备份不等于源码备份，源码仓独立存在是底线。
2. 攒批提交可以，以一个功能模块完整落地为粒度；攒一个多月，历史就会分叉。
3. rsync 同步用 `--checksum`，排除 `.git/`。
4. 公开仓推送前扫敏感信息，分叉了先核对再合并。
