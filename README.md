# 我的个人博客

> 基于 [docsify](https://docsify.js.org/) 构建的轻量级文档 / 博客站点

## 介绍

这是一个使用 **docsify** 搭建的个人知识库与博客。docsify 是一个基于 Markdown 的文档生成工具，它在浏览器运行时动态加载并解析 Markdown 文件，**无需预先生成静态 HTML**，因此部署简单、维护方便。

本项目的主要特性：

- 纯 Markdown 写作，无需任何构建步骤
- 自动生成侧边栏导航与当前页面的标题层级目录
- 内置全文搜索（`search` 插件）
- 支持封面页（coverpage）与顶部导航栏（navbar）
- 核心 JS / CSS 已本地化，可完全离线运行

## 本地预览

```bash
# 在 docsify 目录（index.html 所在目录）下执行
python -m http.server 4000

# 然后在浏览器打开
# http://localhost:4000
```

## 目录结构

```
.
├── index.html        # 站点入口与全局配置（window.$docsify）
├── _sidebar.md       # 侧边栏导航（含页面链接与层级）
├── _navbar.md        # 顶部导航栏
├── _coverpage.md     # 封面页
├── README.md         # 首页（本页）
├── subdoc.md         # 示例文档
├── docs/             # 文档目录
├── subdir/           # 子目录示例
├── static/           # 本地 js / css 资源
└── pic/              # 图片资源
```

## 写作指南

1. 在仓库根目录或任意子目录中新建 / 编辑 `.md` 文件；
2. 在 `_sidebar.md` 中添加对应链接，使其出现在左侧导航；
3. 保存并刷新浏览器即可看到效果（若已部署到 Pages，提交后自动生效）。

## 配置说明

站点行为集中在 `index.html` 的 `window.$docsify` 中：

| 配置项 | 作用 |
| --- | --- |
| `loadSidebar` | 加载 `_sidebar.md` 作为侧边栏 |
| `loadNavbar` | 加载 `_navbar.md` 作为顶部导航 |
| `coverpage` | 启用 `_coverpage.md` 封面页 |
| `subMaxLevel` | 侧边栏中自动生成的标题层级深度 |
| `search` | 搜索插件的提示语与匹配深度 |
| `repo` | 右上角仓库链接 |

## 部署

- **GitHub Pages**：将代码推送至仓库后，在仓库设置中开启 Pages 即可；
- **Gitee Pages**：推送到 Gitee 仓库并开启 Gitee Pages；
- 由于资源均为静态文件，也可直接托管到任意静态服务器 / CDN。

## 参与贡献

1. Fork 本仓库
2. 新建特性分支（`Feat_xxx`）
3. 提交你的修改
4. 发起 Pull Request
