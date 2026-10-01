# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 规则

- 本文件（CLAUDE.md）必须使用中文编写；后续更新时也保持中文。

## 概述

张世林的静态单页个人主页。全部内容都在 `index.html` 中（CSS 与 JS 均内联），无构建步骤、无包管理器、无依赖、无测试。唯一的外部资源是 Google Fonts。`wechet-qr.png`（文件名就是这样拼写的，代码中也这样引用）和 `zalo-qr.jpg` 是联系方式二维码。

## 开发

可直接在浏览器中打开 `index.html`，或启动本地服务器以模拟线上环境：

```sh
python3 -m http.server 8000   # 然后访问 http://localhost:8000
```

没有 lint 或测试工具。复制按钮使用的 `navigator.clipboard` 需要安全上下文，测试时请用 localhost 而不是 `file://`。

## `index.html` 架构

文件顺序：`<style>` → head 中的小段 `<script>` → 页面结构 → `<body>` 末尾的主 IIFE `<script>`。

- **主题**：颜色以 CSS 自定义属性定义在 `:root` 上（默认深色），由 `:root[data-theme="light"]` 覆盖为浅色。切换按钮在 `<html>` 上设置 `data-theme`，并保存到 `localStorage["theme"]`。新增颜色时需在两处都添加变量。
- **多语言（vi / en / zh）**：主脚本中的 `I18N` 对象为每种语言保存一份词典。默认语言是 **`vi`**，尽管 HTML 中写的是中文。`applyLang()` 按属性改写元素：
  - `data-i18n` → `textContent`
  - `data-i18n-html` → `innerHTML`（用于包含 `<strong>` 的文本）
  - `data-i18n-alt` → `alt`，`data-i18n-aria` → `aria-label`
  - `htmlLang` 用于设置 `<html lang>`。所选语言保存到 `localStorage["lang"]`。
  - 新增文本时，必须在**三种语言**的词典中添加相同的 key。HTML 中的中文只是 JS 运行前的兜底内容。
  - `.glitch` 元素会把文本同步到 `data-text`（供 CSS 伪元素使用），因此 `applyLang` 也会刷新该属性。
- **开机动画**：每个会话首次访问时（且未开启减少动态效果），head 脚本会给 `<html>` 加上 `booting` class。主脚本播放终端风格的 `#boot` 日志，随后 `finish()` 设置 `sessionStorage["booted"]`、移除该 class，并启动标语打字效果（`typeTagline`）。点击或按任意键可跳过。
- **视觉效果**：`#rain` canvas 实现"数字雨"，限制在约 20fps，标签页隐藏时暂停。鼠标跟随光晕使用 `--mx`/`--my` 变量（仅限精确指针设备）。`#cardFrame` 有倾斜效果，`#clock` 显示 GMT+8 时钟。
- **减少动态效果**：所有动画都会检查 `prefers-reduced-motion`（JS 中的 `reduceMotion` 变量及 CSS 中对应的 `@media` 块）。新增动画时请保留此判断。
- **联系方式面板**：按钮切换带 `hidden` 的面板（`#wechatPanel`、`#zaloPanel`），同一时间只展开一个。`[data-copy]` 按钮会把该属性值复制到剪贴板。
- **响应式**：断点为 `max-width: 480px` 和 `360px`。

所有 `localStorage`/`sessionStorage` 访问都包裹在 `try/catch` 中，请保持这一写法。
