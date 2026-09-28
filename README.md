# NightFall Blog

个人博客 NightFall（[nightfall7.top](https://nightfall7.top/)）的源码仓库。

## 技术栈

- **静态生成器**：[Hexo](https://hexo.io/) + [Stellar 1.44.0](https://xaoxuu.com/wiki/stellar/) 主题
- **部署**：GitHub Pages，通过 `hexo d` 推送到 [NightNight-30.github.io](https://github.com/NightNight-30/NightNight-30.github.io)
- **图片托管**：阿里云 OSS + CDN，域名 `img.nightfall7.top`
- **评论**：[Artalk](https://artalk.js.org/)

## 目录结构

```
hexo_wsj/
├── source/
│   ├── _posts/        # 文章 Markdown
│   ├── about/         # 关于页
│   ├── friends/       # 友链页
│   ├── life/          # 生活记录
│   ├── shuoshuo/      # 碎碎念
│   ├── css/           # 自定义样式
│   ├── img/           # 小图标（favicon、备案图标等）
│   └── js/            # 自定义脚本
├── _config.yml        # Hexo 配置
└── package.json       # 依赖
```

## 本地开发

```bash
# 安装依赖
npm install

# 本地预览
npx hexo s

# 生成静态文件
npx hexo clean && npx hexo g

# 部署到 GitHub Pages
npx hexo d
```

## 友链

友链数据通过 [NightNight-30/friends](https://github.com/NightNight-30/friends) 仓库的 Issues 管理，GitHub Actions 自动同步。想交换友链请到该仓库提 Issue。
