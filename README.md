# Jackchen Notebook Hexo Blog

这是 `me.jackchen.cn` 的 Hexo 博客源码仓库，文章源码放在 `source/_posts`，静态站点部署到 `jackchensky.github.io` 仓库。

## 本次版本更新

更新时间：2026-06-16 14:53 CST

本次更新主要是为了让旧 Hexo 博客脱离旧版 Node 和旧版 NexT 主题依赖，后续写文章、预览和部署都能在较新的 Node 环境下稳定运行。

### 更新内容

- 恢复线上缺失文章源码：补回 `source/_posts/2021-9-qinglong bulid.md`，让本地源码和线上文章内容保持一致。
- 升级 Hexo 运行环境：从旧版 Hexo 3.x 升级到 Hexo `8.1.2`，并将 Node 要求固定为 `>=20.19.0`。
- 更新 Hexo 插件依赖：升级生成器、渲染器、Feed、Sitemap、部署插件等依赖，并新增 `clean`、`build`、`server`、`deploy` 常用脚本。
- 升级 NexT 主题：从本地旧主题 NexT `5.1.4` 迁移到 npm 依赖 `hexo-theme-next@8.27.0`。
- 迁移主题配置：新增 `_config.next.yml` 管理 NexT 8 配置，保留旧主题目录到 `themes/next-legacy` 作为回退参考。
- 移除旧模板依赖：新版 NexT 使用 Nunjucks，不再需要旧的 `hexo-renderer-swig`。
- 更新站点配置：站点 URL 调整为 `https://me.jackchen.cn`，语言配置调整为 `zh-CN`。
- 更新导航菜单：顶部菜单中的 `Blog` 指向 `https://jackchen.cn`。
- 完成线上发布：已生成并部署到 GitHub Pages，线上域名 `https://me.jackchen.cn` 已验证可访问新版 NexT 页面。

### 验证记录

- `npm run clean` 通过。
- `npm run build` 通过，生成日志显示 `NexT version 8.27.0`。
- 本地预览验证过首页、分页、归档、分类、标签、文章页、`atom.xml` 和 `sitemap.xml`。
- 线上验证过首页、分页、归档、分类、标签和文章页均返回 `200`，并确认使用 NexT `8.27.0`。

## 常用命令

```bash
npm install
npm run clean
npm run build
npm run server
npm run deploy
```

## 主要文件

- `_config.yml`：Hexo 站点配置。
- `_config.next.yml`：NexT 8 主题配置。
- `source/_posts`：博客文章 Markdown 源文件。
- `themes/next-legacy`：旧 NexT 5.1.4 主题备份，仅作参考。
- `public`：Hexo 生成的静态文件目录。
