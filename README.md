# hyblog

基于 Hugo + PaperMod 的静态博客，部署在 GitHub Pages。

## 工作流程（只需写 Markdown）

本仓库是纯静态站点：**你只需要写 Markdown 文章并推送**，GitHub Actions 会自动构建并部署到 https://hhhheying.github.io/。

### 本机准备

- 安装 Git 和任意 Markdown 编辑器（VS Code / Typora 等）。
- **不需要安装 Hugo**。

### 写一篇新文章

1. 克隆/拉取仓库：

   ```bash
   git clone https://github.com/hhhheying/hhhheying.github.io.git
   ```

2. 在 `content/posts/` 下新建 Markdown 文件（文件名用英文，如 `my-first-post.md`），front matter 参考：

   ```markdown
   ---
   title: "文章标题"
   date: 2024-01-01T00:00:00+08:00
   lastmod: 2024-01-01T00:00:00+08:00
   description: "文章简介"
   tags: ["标签一", "标签二"]
   categories: ["分类"]
   ---

   正文……
   ```

3. 提交并推送，等待 Actions 构建（约 1 分钟）后即可在线上看到：

   ```bash
   git add .
   git commit -m "add post: my-first-post"
   git push
   ```

## 常用功能

- **目录跳转**：文章内的标题会自动生成右侧/顶部目录，可点击跳转。
- **标签/分类**：在 front matter 中声明 `tags` / `categories`，自动生成列表页。
- **深色模式**：右上角太阳/月亮图标切换，默认跟随系统。
- **归档**：`/archives/` 按日期归档全部文章。
- **RSS**：`/index.xml`。

## 构建说明

- 构建流程见 `.github/workflows/hugo.yml`，Hugo 版本为 `0.128.0`。
- 若要迁移构建方式（如换成其他 CI 或本机构建），只需修改该 workflow，或在装了 Hugo 的机器上执行 `hugo`（需先删掉旧构建产物 `public/`）。

## 旧版 Hexo 博客

旧版 Hexo 站已备份到 `old-hexo` 分支，需要时：

```bash
git checkout old-hexo
```
