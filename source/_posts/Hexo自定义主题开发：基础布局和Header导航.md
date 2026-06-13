---
title: Hexo自定义主题开发：1、基础布局和Header导航
date: 2026-06-13 19:15:22
tags:
 -学习
 -主题
category: 主题开发
---
## 一、前言

本文基于 Hexo 官方主题开发文档，从零搭建自定义 Hexo 主题的基础架构，完成主题目录初始化、全局基础布局、自适应 Header 导航栏开发。同时配置深浅色主题切换、全局样式规范、响应式适配，打造一套美观、可扩展、带霓虹特效的博客基础页面框架。

官方参考文档：[Hexo主题官方文档](https://hexo.io/zh-cn/docs/themes)

## 二、自定义主题目录初始化

根据 Hexo 主题开发规范，所有自定义主题均存放在博客根目录的 `themes` 文件夹下。首先在 `my_blog/themes/` 目录中创建自定义主题文件夹 `my_theme`，并搭建官方标准主题目录结构，各目录作用遵循 Hexo 内核规则：

- **\_config\.yml**：主题独立配置文件，修改后无需重启 Hexo 服务，自动热更新生效

- **languages**：国际化语言配置文件夹，支持多语言博客适配

- **layout**：模板布局文件夹，存放页面结构模板（本文使用 EJS 模板）

- **scripts**：脚本文件夹，Hexo 启动时自动加载内部 JS 文件

- **source**：静态资源文件夹，存放 CSS、JS、图片等素材，下划线开头/隐藏文件会被 Hexo 忽略

### 2\.1 完整目录结构

```plain
my_theme
    ├── _config.yml      # 主题配置文件
    ├── languages        # 多语言配置目录
    ├── layout           # 页面模板布局目录
    ├── scripts          # 自定义脚本目录
    └── source           # 静态资源目录（CSS/JS/图片）
```

### 2\.2 启用自定义主题

修改博客根目录的 `_config.yml` 全局配置文件，将主题切换为我们新建的 `my_theme`：

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: my_theme
```

此时主题已成功启用，但暂无任何页面内容，页面为空白。接下来我们逐步搭建页面布局与样式。

## 三、基础模板文件搭建

Hexo 支持模板文件拆分复用，我们将页面头部、头部资源单独封装，实现代码解耦，方便后续维护迭代。

### 3\.1 封装头部导航模板（header\.ejs）

在 `my_theme/layout/_partial/`路径下新建 `header.ejs`，作为全局共用的导航头部，包含网站 Logo、桌面端导航菜单、移动端菜单按钮，预留后续变量动态配置接口。

```html
<header class="site-header">
  <div class="container header-inner">
    <div class="logo">
      NULL VOID
    </div>
    <div class="header-right">
      <nav class="nav-menu" id="navMenu">
        <a href="/" class="active">首页</a>
        <a href="/">归档</a>
        <a href="/">分类</a>
        <a href="/">标签</a>
        <a href="/">关于</a>
      </nav>
      <button class="menu-toggle" id="mobileMenuBtn" aria-label="菜单">
        <i class="fas fa-bars"></i>
      </button>
    </div>
  </div>
</header>
```

### 3\.2 封装头部资源模板（head\.ejs）

为避免主模板代码冗余，将页面头部资源、字体、图标、样式引入单独封装为 `head.ejs`，统一管理全局依赖，包含：自定义字体、FontAwesome 图标库、全局基础样式文件。

**补充说明**：Google 字体链接若加载失败，可替换为国内镜像地址，解决网络访问问题。

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=yes">
<title><%= config.title %> | <%= config.subtitle || '绝区零风格博客' %></title>
<!-- 全局自定义字体 -->
<link href="https://fonts.googleapis.com/css2?family=Inter:opsz,wght@14..32,300;14..32,400;14..32,600;14..32,700;14..32,800&family=Space+Grotesk:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
<!-- FontAwesome 图标库 -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
<!-- 自定义分层CSS样式 -->
<link rel="stylesheet" href="<%- url_for('/css/base/variables.css') %>">
<link rel="stylesheet" href="<%- url_for('/css/base/reset.css') %>">
<link rel="stylesheet" href="<%- url_for('/css/layout/grid.css') %>">
<link rel="stylesheet" href="<%- url_for('/css/layout/header.css') %>">
```

### 3\.3 主布局模板（layout\.ejs）

创建主题核心主布局文件，引入头部资源和导航栏，同时适配深浅色主题切换逻辑，是全站页面的基础骨架。

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <%- partial('_partial/head') %>
</head>
<body class="<%= (config.theme_config && config.theme_config.default_theme === 'light') ? 'light-theme' : '' %>">
  <%- partial('_partial/header') %>
</body>
</html>
```

## 四、全局样式体系开发

为保证样式可维护、解耦，我们采用**分层CSS架构**，分为：全局变量、样式重置、网格布局、导航样式四大模块，同时适配深色/浅色双主题，自带霓虹发光、动态动画、响应式适配效果。

### 4\.1 全局变量样式（variables\.css）

统一管理全站颜色、阴影、动画、间距、主题适配变量，后续修改全站样式仅需调整此处，大幅提升维护效率。包含深色默认主题、浅色主题两套变量覆盖规则。

```css
/* CSS 自定义属性（颜色、阴影、动画时长、布局参数全局统一管理） */
:root {
    --bg-dark: #05070a;
    --bg-card: rgba(12, 16, 24, 0.75);
    --bg-elevated: #0e121c;
    --primary-neon: #ff2a7e;
    --secondary-neon: #00f2ff;
    --accent-purple: #b100e8;
    --accent-amber: #ff6b35;
    --text-main: #edeef2;
    --text-dim: #9ca3af;
    --border-glow: rgba(255, 42, 126, 0.25);
    --grid-color: rgba(0, 242, 255, 0.08);
    --card-border: rgba(255, 255, 255, 0.08);
    --btn-hover-bg: var(--primary-neon);
    --btn-hover-text: #0a0c10;
    --transition-theme: background-color 0.3s ease, color 0.2s ease, border-color 0.2s ease, box-shadow 0.2s ease;
    --menu-bg: rgba(5, 7, 10, 0.98);
    --deco-glow-1: rgba(255, 42, 126, 0.12);
    --deco-glow-2: rgba(0, 242, 255, 0.09);
    --scanline-opacity: 0.04;
    --particle-color-1: rgba(255, 42, 126, 0.55);
    --particle-color-2: rgba(0, 242, 255, 0.5);
    --particle-color-3: rgba(177, 0, 232, 0.4);
    --logo-gradient-dark: #ffffff;
    --hero-gradient-dark: #ffffff;
    --card-img-bg: #0a0f18;
    --card-tag-bg: rgba(0, 0, 0, 0.75);
    --card-tag-text: #00f2ff;
    --mobile-menu-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.6);
    --postcard-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
    --postcard-hover-shadow: 0 0 30px rgba(255, 42, 126, 0.2), 0 0 60px rgba(0, 242, 255, 0.08), 0 16px 32px rgba(0, 0, 0, 0.4);
    --sidecard-hover-shadow: 0 0 20px rgba(255, 42, 126, 0.08);
    --header-shadow: 0 4px 30px rgba(0, 0, 0, 0.3);
    --readmore-text-shadow: 0 0 4px rgba(0, 242, 255, 0.3);
    --readmore-hover-text-shadow: 0 0 8px rgba(255, 42, 126, 0.5);
    --btn-neon-shadow: 0 0 12px rgba(255, 42, 126, 0.15);
    --btn-neon-hover-shadow: 0 0 24px var(--primary-neon), 0 0 48px rgba(255, 42, 126, 0.4);
    --action-btn-shadow: 0 0 16px rgba(0, 0, 0, 0.4);
    --action-btn-hover-shadow: 0 0 22px var(--primary-neon), 0 8px 20px rgba(0, 0, 0, 0.4);
    --menu-toggle-shadow: 0 0 8px rgba(255, 42, 126, 0.2);
    --menu-toggle-hover-shadow: 0 0 16px rgba(255, 42, 126, 0.4);
    --tag-cloud-bg: rgba(0, 242, 255, 0.08);
    --tag-cloud-border: rgba(0, 242, 255, 0.2);
    --tag-cloud-hover-bg: #ff2a7e;
    --tag-cloud-hover-shadow: 0 0 14px #ff2a7e;
    --recent-list-border: rgba(255, 255, 255, 0.1);
    --skew-decoration-bg: rgba(255, 42, 126, 0.08);
    --skew-decoration2-bg: radial-gradient(circle at 30% 35%, rgba(0, 242, 255, 0.12) 0%, rgba(0, 242, 255, 0.04) 35%, transparent 60%), radial-gradient(circle at 50% 55%, rgba(177, 0, 232, 0.07) 0%, transparent 50%), radial-gradient(circle at 25% 50%, rgba(255, 42, 126, 0.05) 0%, transparent 45%);
    --cursor-glow-bg: radial-gradient(circle at center, rgba(255, 42, 126, 0.07) 0%, rgba(0, 242, 255, 0.04) 30%, transparent 65%);
}

/* 浅色主题变量覆盖 */
body.light-theme {
    --bg-dark: #f5f3f0;
    --bg-card: rgba(255, 255, 255, 0.88);
    --bg-elevated: #ffffff;
    --primary-neon: #d4193e;
    --secondary-neon: #0b6e7a;
    --accent-purple: #9b1ab0;
    --accent-amber: #c25528;
    --text-main: #1a1d28;
    --text-dim: #545a68;
    --border-glow: rgba(212, 25, 62, 0.28);
    --grid-color: rgba(11, 110, 122, 0.06);
    --card-border: rgba(0, 0, 0, 0.1);
    --btn-hover-bg: var(--primary-neon);
    --btn-hover-text: #ffffff;
    --menu-bg: rgba(245, 243, 240, 0.98);
    --deco-glow-1: rgba(212, 25, 62, 0.05);
    --deco-glow-2: rgba(11, 110, 122, 0.04);
    --scanline-opacity: 0.015;
    --particle-color-1: rgba(212, 25, 62, 0.35);
    --particle-color-2: rgba(11, 110, 122, 0.32);
    --particle-color-3: rgba(155, 26, 176, 0.25);
    --logo-gradient-dark: #1a1d28;
    --hero-gradient-dark: #1a1d28;
    --card-img-bg: #e8e5e0;
    --card-tag-bg: rgba(255, 255, 255, 0.85);
    --card-tag-text: #0b6e7a;
    --mobile-menu-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.1);
    --postcard-shadow: 0 6px 18px rgba(0, 0, 0, 0.06);
    --postcard-hover-shadow: 0 0 18px rgba(212, 25, 62, 0.14), 0 0 40px rgba(11, 110, 122, 0.05), 0 14px 28px rgba(0, 0, 0, 0.1);
    --sidecard-hover-shadow: 0 0 16px rgba(212, 25, 62, 0.06);
    --header-shadow: 0 4px 30px rgba(0, 0, 0, 0.07);
    --readmore-text-shadow: 0 0 3px rgba(11, 110, 122, 0.2);
    --readmore-hover-text-shadow: 0 0 6px rgba(212, 25, 62, 0.35);
    --btn-neon-shadow: 0 0 10px rgba(212, 25, 62, 0.12);
    --btn-neon-hover-shadow: 0 0 20px rgba(212, 25, 62, 0.35), 0 0 36px rgba(212, 25, 62, 0.2);
    --action-btn-shadow: 0 0 12px rgba(0, 0, 0, 0.12);
    --action-btn-hover-shadow: 0 0 18px rgba(212, 25, 62, 0.3), 0 8px 16px rgba(0, 0, 0, 0.12);
    --menu-toggle-shadow: 0 0 6px rgba(212, 25, 62, 0.15);
    --menu-toggle-hover-shadow: 0 0 12px rgba(212, 25, 62, 0.3);
    --tag-cloud-bg: rgba(11, 110, 122, 0.06);
    --tag-cloud-border: rgba(11, 110, 122, 0.2);
    --tag-cloud-hover-bg: #d4193e;
    --tag-cloud-hover-shadow: 0 0 12px rgba(212, 25, 62, 0.35);
    --recent-list-border: rgba(0, 0, 0, 0.08);
    --skew-decoration-bg: rgba(212, 25, 62, 0.04);
    --skew-decoration2-bg: radial-gradient(circle at 30% 35%, rgba(11, 110, 122, 0.07) 0%, rgba(11, 110, 122, 0.02) 35%, transparent 60%), radial-gradient(circle at 50% 55%, rgba(155, 26, 176, 0.03) 0%, transparent 50%), radial-gradient(circle at 25% 50%, rgba(212, 25, 62, 0.025) 0%, transparent 45%);
    --cursor-glow-bg: radial-gradient(circle at center, rgba(212, 25, 62, 0.04) 0%, rgba(11, 110, 122, 0.025) 30%, transparent 65%);
}
```

### 4\.2 全局样式重置（reset\.css）

重置浏览器默认样式，统一盒模型、滚动条、页面基础样式，消除不同浏览器的样式差异，保证全站样式统一性。

```css
/* 浏览器默认样式重置（统一盒模型、全局基础样式） */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    background-color: var(--bg-dark);
    background-image:
        repeating-linear-gradient(45deg, var(--grid-color) 0px, var(--grid-color) 2px, transparent 2px, transparent 12px),
        radial-gradient(circle at 15% 30%, var(--deco-glow-1) 0%, transparent 40%),
        radial-gradient(circle at 85% 70%, var(--deco-glow-2) 0%, transparent 45%);
    color: var(--text-main);
    font-family: 'Inter', 'Space Grotesk', system-ui, -apple-system, sans-serif;
    line-height: 1.5;
    scroll-behavior: smooth;
    min-height: 100vh;
    transition: var(--transition-theme);
    cursor: crosshair;
    overflow-x: hidden;
    position: relative;
}

body.light-theme {
    background-image:
        repeating-linear-gradient(45deg, var(--grid-color) 0px, var(--grid-color) 2px, transparent 2px, transparent 12px),
        radial-gradient(circle at 15% 30%, var(--deco-glow-1) 0%, transparent 40%),
        radial-gradient(circle at 85% 70%, var(--deco-glow-2) 0%, transparent 45%);
}

/* 自定义滚动条样式 */
::-webkit-scrollbar {
    width: 6px;
}

::-webkit-scrollbar-track {
    background: #0a0c12;
}

body.light-theme ::-webkit-scrollbar-track {
    background: #e2e5ea;
}

::-webkit-scrollbar-thumb {
    background: var(--primary-neon);
    border-radius: 8px;
    box-shadow: 0 0 6px var(--primary-neon);
}
```

### 4\.3 全局网格布局（grid\.css）

搭建博客核心布局体系，包含容器、双栏博客布局、卡片网格、通用标题样式，同时完成移动端、平板、手机端全响应式适配。

```css
/* 网格系统、博客全局主布局样式 */
.container {
    max-width: 1400px;
    margin: 0 auto;
    padding: 0 24px;
    position: relative;
    z-index: 2;
}

/* 博客主体双栏布局：内容区 + 侧边栏 */
.blog-grid {
    display: grid;
    grid-template-columns: 1fr 320px;
    gap: 40px;
    margin: 0 0 60px;
    align-items: flex-start;
}

/* 文章卡片自适应网格 */
.card-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 28px;
}

main {
    position: relative;
    z-index: 2;
}

/* 通用区块标题样式 */
.section-title {
    font-size: 1.8rem;
    margin-bottom: 28px;
    display: inline-block;
    letter-spacing: -0.3px;
    color: var(--text-main);
    position: relative;
    padding-bottom: 8px;
}

.section-title i {
    color: var(--primary-neon);
    margin-right: 10px;
    animation: iconFlicker 3s ease-in-out infinite;
}

.section-title::after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 0;
    width: 50%;
    height: 2px;
    background: linear-gradient(90deg, var(--primary-neon), transparent);
    box-shadow: 0 0 6px var(--primary-neon);
}

body.light-theme .section-title::after {
    box-shadow: 0 0 3px var(--primary-neon);
}

/* 标题图标闪烁动画 */
@keyframes iconFlicker {
    0%,
    100% {
        opacity: 1;
        text-shadow: 0 0 6px var(--primary-neon);
    }
    30% {
        opacity: 0.7;
        text-shadow: 0 0 12px var(--primary-neon);
    }
    60% {
        opacity: 1;
        text-shadow: 0 0 4px var(--primary-neon);
    }
}

/* 响应式适配 - 平板设备 */
@media (max-width: 768px) {
    .container {
        padding: 0 18px;
    }

    .section-title {
        font-size: 1.5rem;
        margin-bottom: 20px;
    }

    .blog-grid {
        grid-template-columns: 1fr;
        gap: 36px;
    }

    .card-grid {
        grid-template-columns: 1fr;
    }
}

/* 响应式适配 - 手机设备 */
@media (max-width: 480px) {
    .container {
        padding: 0 14px;
    }
}
```

### 4\.4 顶部导航样式（header\.css）

实现导航栏粘性置顶、毛玻璃磨砂背景、霓虹发光特效、hover 动态下划线、移动端折叠菜单，适配深浅色主题切换。

```css
/* 顶部导航栏专属样式 */
.site-header {
    padding: 20px 0;
    position: sticky;
    top: 0;
    z-index: 1000;
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    background: rgba(5, 7, 10, 0.7);
    border-bottom: 1px solid var(--border-glow);
    transition: var(--transition-theme);
    box-shadow: var(--header-shadow);
}

body.light-theme .site-header {
    background: rgba(245, 243, 240, 0.82);
    box-shadow: var(--header-shadow);
    border-bottom-color: rgba(0, 0, 0, 0.08);
}

.header-inner {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: nowrap;
    position: relative;
}

.header-right {
    display: flex;
    align-items: center;
    gap: 16px;
    margin-left: auto;
}

/* 网站Logo样式 */
.logo {
    display: flex;
    align-items: baseline;
    gap: 8px;
    font-family: 'Space Grotesk', monospace;
    font-weight: 700;
    font-size: 1.8rem;
    letter-spacing: -0.02em;
    background: linear-gradient(135deg, var(--logo-gradient-dark) 20%, var(--secondary-neon) 60%, var(--primary-neon));
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
    text-shadow: 0 0 12px rgba(0, 242, 255, 0.4), 0 0 30px rgba(255, 42, 126, 0.25);
    transition: var(--transition-theme);
    flex-shrink: 0;
    position: relative;
    animation: logoPulse 4s ease-in-out infinite;
}

body.light-theme .logo {
    text-shadow: 0 0 6px rgba(11, 110, 122, 0.25), 0 0 16px rgba(212, 25, 62, 0.15);
}

.logo span {
    font-size: 1rem;
    background: none;
    -webkit-background-clip: unset;
    background-clip: unset;
    color: var(--secondary-neon);
    font-weight: 400;
    white-space: nowrap;
    text-shadow: 0 0 6px rgba(0, 242, 255, 0.3);
}

body.light-theme .logo span {
    text-shadow: 0 0 4px rgba(11, 110, 122, 0.2);
}

.logo::before {
    content: '';
    position: absolute;
    left: -14px;
    top: 50%;
    transform: translateY(-50%);
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: var(--primary-neon);
    box-shadow: 0 0 10px var(--primary-neon), 0 0 20px var(--primary-neon);
    animation: statusBlink 2s ease-in-out infinite;
}

/* Logo呼吸闪烁动画 */
@keyframes logoPulse {
    0%,
    100% {
        filter: brightness(1);
    }
    50% {
        filter: brightness(1.2);
    }
}

/* 状态点呼吸动画 */
@keyframes statusBlink {
    0%,
    100% {
        opacity: 1;
        box-shadow: 0 0 10px var(--primary-neon), 0 0 20px var(--primary-neon);
    }
    50% {
        opacity: 0.3;
        box-shadow: 0 0 4px var(--primary-neon), 0 0 8px var(--primary-neon);
    }
}

/* 桌面端导航菜单 */
.nav-menu {
    display: flex;
    gap: 2rem;
    align-items: center;
    transition: var(--transition-theme);
}

.nav-menu a {
    color: var(--text-dim);
    text-decoration: none;
    font-weight: 500;
    font-size: 1rem;
    transition: var(--transition-theme);
    position: relative;
    letter-spacing: 0.3px;
    white-space: nowrap;
    padding: 4px 0;
}

.nav-menu a:hover,
.nav-menu a.active {
    color: var(--secondary-neon);
    text-shadow: 0 0 8px rgba(0, 242, 255, 0.6), 0 0 16px rgba(0, 242, 255, 0.3);
}

body.light-theme .nav-menu a:hover,
body.light-theme .nav-menu a.active {
    text-shadow: 0 0 4px rgba(11, 110, 122, 0.35), 0 0 10px rgba(11, 110, 122, 0.18);
}

.nav-menu a::after {
    content: '';
    position: absolute;
    bottom: -6px;
    left: 0;
    width: 0;
    height: 2px;
    background: linear-gradient(90deg, var(--primary-neon), var(--secondary-neon));
    transition: width 0.3s cubic-bezier(0.22, 0.61, 0.36, 1);
    box-shadow: 0 0 8px var(--primary-neon);
}

body.light-theme .nav-menu a::after {
    box-shadow: 0 0 4px var(--primary-neon);
}

.nav-menu a:hover::after,
.nav-menu a.active::after {
    width: 100%;
}

/* 移动端菜单按钮 */
.menu-toggle {
    display: none;
    background: none;
    border: 1px solid var(--primary-neon);
    font-size: 1.5rem;
    color: var(--secondary-neon);
    cursor: pointer;
    padding: 8px 12px;
    border-radius: 8px;
    transition: var(--transition-theme);
    flex-shrink: 0;
    box-shadow: var(--menu-toggle-shadow);
}

.menu-toggle:hover {
    background: rgba(255, 42, 126, 0.2);
    box-shadow: var(--menu-toggle-hover-shadow);
}

body.light-theme .menu-toggle:hover {
    background: rgba(212, 25, 62, 0.1);
}

/* 移动端导航适配 */
@media (max-width: 768px) {
    .menu-toggle {
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .header-right {
        gap: 10px;
    }

    .site-header {
        position: sticky;
    }

    .header-inner {
        position: relative;
        flex-wrap: nowrap;
    }

    /* 移动端折叠导航菜单 */
    .nav-menu {
        position: absolute;
        top: calc(100% + 8px);
        left: 0;
        right: 0;
        background: var(--menu-bg);
        backdrop-filter: blur(20px);
        -webkit-backdrop-filter: blur(20px);
        border-radius: 20px;
        border: 1px solid var(--primary-neon);
        box-shadow: var(--mobile-menu-shadow);
        padding: 1.2rem 1.5rem;
        flex-direction: column;
        gap: 1.2rem;
        align-items: stretch;
        z-index: 1050;
        opacity: 0;
        visibility: hidden;
        transform: translateY(-12px);
        transition: opacity 0.25s ease, visibility 0.25s, transform 0.25s ease;
        pointer-events: none;
        max-height: calc(100vh - 100px);
        overflow-y: auto;
        display: flex;
    }

    body.light-theme .nav-menu {
        border-color: rgba(212, 25, 62, 0.5);
        box-shadow: var(--mobile-menu-shadow);
    }

    .nav-menu.active {
        opacity: 1;
        visibility: visible;
        transform: translateY(0);
        pointer-events: auto;
    }

    .nav-menu a {
        font-size: 1.1rem;
        padding: 8px 0;
        display: block;
        text-align: center;
        border-bottom: 1px solid var(--border-glow);
    }

    body.light-theme .nav-menu a {
        border-bottom-color: rgba(0, 0, 0, 0.08);
    }

    .nav-menu a:last-child {
        border-bottom: none;
    }

    .logo {
        font-size: 1.5rem;
    }
}

@media (max-width: 480px) {
    .header-right {
        gap: 6px;
    }
}
```

## 五、开发总结与后续拓展

### 5\.1 本期完成内容

- 搭建 Hexo 官方规范的自定义主题完整目录结构，完成主题启用配置

- 封装可复用的 EJS 模板（头部资源、全局导航、主布局骨架）

- 搭建分层、可维护的 CSS 样式架构，统一全局样式规范

- 实现深色/浅色双主题无缝切换，自带霓虹发光、呼吸动画、悬浮特效

- 完成桌面、平板、手机三端全响应式适配，包含移动端折叠导航

- 自定义滚动条、网格背景、动态标题、Logo 特效等视觉美化

### 5\.2 后续可拓展方向

- 编写 JS 逻辑，实现移动端菜单点击展开/收起、主题切换按钮功能

- 封装首页、文章页、归档页、分类页等专属模板

- 添加文章卡片、侧边栏、页脚、版权信息等页面模块

- 接入搜索、评论、阅读统计等 Hexo 插件，丰富博客功能

- 优化静态资源加载，添加懒加载、CDN 适配，提升页面加载速度

> （注：文档部分内容可能由 AI 生成）
