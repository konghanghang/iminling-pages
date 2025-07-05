# 记录点滴 - 个人技术博客

基于Hugo和Stack主题构建的个人技术博客，记录日常学习和实践过程中的点点滴滴。

## 📖 项目简介

本项目是一个使用Hugo静态网站生成器搭建的个人技术博客，采用了现代化的Stack主题，专注于技术文章的分享与交流。

## ✨ 特性

- 🚀 **高性能**：基于Hugo的静态网站生成
- 📱 **响应式设计**：适配各种设备屏幕
- 🔍 **全文搜索**：支持站内文章搜索
- 📚 **文章引用**：智能的文章相互引用系统
- 🎨 **美观界面**：现代化的UI设计
- 🌍 **SEO优化**：搜索引擎友好
- 📊 **站点地图**：自动生成优化的sitemap

## 🚀 快速开始

### 环境要求

- Hugo Extended >= 0.147.0
- Node.js >= 16.0
- Git

### 安装步骤

1. **克隆项目**
   ```bash
   git clone https://github.com/your-username/iminling-pages.git
   cd iminling-pages
   ```

2. **安装主题（重要！）**
   
   **方式一：使用 Git Submodules（推荐）**
   ```bash
   # 初始化并更新子模块
   git submodule update --init --recursive
   ```
   
   **方式二：手动克隆主题**
   ```bash
   # 如果子模块方式失败，可以手动克隆主题
   git clone https://github.com/CaiJimmy/hugo-theme-stack.git themes/hugo-theme-stack
   ```

   > ⚠️ **重要提醒**：本项目使用 Hugo Stack 主题，首次运行前必须确保 `themes/hugo-theme-stack` 目录存在且包含主题文件，否则会出现 "Page Not Found" 错误。

3. **本地运行**
   ```bash
   hugo server -D
   ```

4. **访问网站**
   打开浏览器访问 `http://localhost:1313`

### 主题管理

本项目使用 Git Submodules 来管理主题，这样可以：

- 🔄 **版本控制**：主题版本与项目版本分离管理
- 📦 **依赖管理**：自动处理主题依赖关系
- 🚀 **快速部署**：一键更新主题到最新版本

#### 更新主题
```bash
# 更新主题到最新版本
git submodule update --remote themes/hugo-theme-stack
```

#### 切换主题版本
```bash
# 切换到特定版本
cd themes/hugo-theme-stack
git checkout v4.0.0
cd ../..
```

## 📚 文章引用功能

本项目实现了强大的文章引用系统，支持多种引用方式：

### 1. 基础引用方法

#### 使用 `ref` shortcode（推荐）
```markdown
{{</* ref "/post/path/to/article.md" */>}}
```

#### 使用 `relref` shortcode
```markdown
{{</* relref "/post/path/to/article.md" */>}}
```

#### 直接 Markdown 链接
```markdown
[文章标题](/post/article-slug/)
```

### 2. 高级引用功能

#### 引用特定章节
```markdown
{{</* ref "/post/article.md#section-title" */>}}
```

#### 自定义引用卡片
```markdown
{{</* post-ref "/post/wordpress/WordPress使用git-it-write插件配合github自动发布markdown.md" */>}}
```

### 3. 引用卡片样式

项目包含了自定义的引用卡片样式，提供美观的文章引用展示：

- 📄 **文章标题**：清晰显示被引用文章的标题
- 📝 **文章摘要**：自动截取文章摘要内容
- 📅 **发布日期**：显示文章发布时间
- 📁 **分类信息**：显示文章所属分类
- 🎨 **悬停效果**：鼠标悬停时的动画效果

## 🗂️ 项目结构

```
iminling-pages/
├── content/              # 内容目录
│   ├── post/            # 文章目录
│   │   ├── hugo/        # Hugo相关文章
│   │   ├── wordpress/   # WordPress相关文章
│   │   ├── linux/       # Linux相关文章
│   │   ├── tools/       # 工具类文章
│   │   ├── java/        # Java相关文章
│   │   └── ...          # 其他分类
│   ├── page/            # 页面目录
│   │   ├── about/       # 关于页面
│   │   ├── archives/    # 归档页面
│   │   ├── links/       # 友链页面
│   │   └── search/      # 搜索页面
│   └── _index.md        # 首页内容
├── layouts/             # 布局模板
│   ├── shortcodes/      # 短代码
│   │   └── post-ref.html # 文章引用卡片
│   └── sitemap.xml      # 自定义sitemap模板
├── assets/              # 资源文件
│   └── scss/
│       └── custom.scss  # 自定义样式
├── static/              # 静态文件
│   └── robots.txt       # 搜索引擎爬虫配置
├── themes/              # 主题目录
│   └── hugo-theme-stack/
├── hugo.yaml            # Hugo配置文件
└── README.md            # 项目说明文档
```

## ⚙️ 配置说明

### 基础配置

主要配置文件：`hugo.yaml`

```yaml
baseurl: https://www.iminling.com/
languageCode: zh-cn
theme: hugo-theme-stack
title: 记录点滴 - 分享
hasCJKLanguage: true
```

### Sitemap配置

```yaml
sitemap:
  changefreq: weekly
  priority: 0.5
  filename: sitemap.xml
```

### 相关文章配置

```yaml
related:
  includeNewer: true
  threshold: 60
  toLower: false
  indices:
    - name: tags
      weight: 100
    - name: categories
      weight: 200
```

## 🔧 自定义功能

### 1. 优化的Sitemap

- 自动排除分类和标签页面
- 只包含实际内容页面
- 提高SEO效果

### 2. 文章引用卡片

- 自定义shortcode：`post-ref`
- 美观的卡片式展示
- 自动提取文章信息

### 3. 响应式样式

- 适配移动设备
- 暗黑模式支持
- 现代化UI设计

## 🚀 构建与部署

### 本地构建

```bash
# 清理并构建
hugo --cleanDestinationDir

# 构建到指定目录
hugo --destination ./public
```

### 部署到GitHub Pages

```bash
# 构建静态文件
hugo --minify

# 部署到GitHub Pages
git add .
git commit -m "Update site"
git push origin main
```

### 部署到其他平台

项目支持部署到多个平台：
- GitHub Pages
- Netlify
- Vercel
- 服务器

## 📝 写作指南

### 文章模板

```yaml
---
title: "文章标题"
date: 2025-01-07
categories:
  - 分类名称
tags:
  - 标签1
  - 标签2
---

# 文章内容

这里是文章的正文内容...

## 相关文章

{{</* post-ref "/post/related-article.md" */>}}
```

### 最佳实践

1. **文件命名**：使用有意义的文件名
2. **目录组织**：按分类组织文章到不同目录
3. **标签使用**：合理使用标签和分类
4. **文章引用**：善用引用功能建立文章间的联系

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'Add amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 打开 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 🙏 致谢

- [Hugo](https://gohugo.io/) - 强大的静态网站生成器
- [Stack Theme](https://github.com/CaiJimmy/hugo-theme-stack) - 优秀的Hugo主题
- 所有贡献者和用户

## 📞 联系方式

- 网站：[https://www.iminling.com](https://www.iminling.com)
- 邮箱：[your-email@example.com](mailto:your-email@example.com)
- GitHub：[@your-username](https://github.com/your-username)

---

⭐ 如果这个项目对您有帮助，请给个星星！