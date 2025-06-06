# 引言

[[toc]]

::: tip

使用 [Grapes Studio SDK](https://app.grapesjs.com/docs-sdk/overview/getting-started) 为您的网页构建器(web builder)注入强大动力——它提供可定制的 GrapesJS 体验，并配备了精美的用户界面(UI)，可随时嵌入您的应用中。

:::


## GrapesJS 是什么？

乍一看，您可能会认为这只是又一个页面/HTML 构建器，但它远不止于此。GrapesJS 是一个多功能的网页构建器框架(Web Builder Framework)，这意味着它能让您轻松地为各种“事物”创建支持拖放功能的构建器。这里的“事物”指的是任何具有类 HTML 结构的内容，这远不止网页。事实上，我们几乎在所有地方都使用类 HTML 结构：例如，在电子邮件通讯 (如 [MJML](https://mjml.io/))、原生移动应用 (如 [React Native](https://github.com/facebook/react-native))、原生桌面应用 (如 [Vuido](https://vuido.mimec.org))、PDF (如 [React PDF](https://github.com/diegomura/react-pdf)) 等等。因此，对于任何您可以想象到的、由 `<tag some="attribute">... 其他嵌套元素 ...</tag>` 这样的元素集合构成的内容，您都可以轻松地围绕它创建一个 GrapesJS 构建器，并在您的应用程序中独立使用。
GrapesJS 自带了多种功能和工具，使您能够打造出简单易用的构建器。这让您的用户即使不具备任何编程知识，也能创建出复杂的类 HTML 模板。

## 为什么选择 GrapesJS？

GrapesJS 的主要设计目的是在内容管理系统(Content Management Systems)中使用，以加速动态模板的创建过程，并取代常见的所见即所得(WYSIWYG)编辑器。这些编辑器虽然适合编辑内容，但并不适合创建 HTML 结构。我们没有选择创建一个封闭的应用程序，而是决定打造一个可扩展的框架，让任何人都可以将其用于任何目的。

## 快速入门

为了展示 GrapesJS 的强大功能，我们创建了一些预设(presets)。

- [grapesjs-preset-webpage](https://github.com/GrapesJS/preset-webpage) - [网页构建器演示](https://grapesjs.com/demo.html)
- [grapesjs-preset-newsletter](https://github.com/GrapesJS/preset-newsletter) - [邮件通讯构建器(Newsletter Builder)演示](https://grapesjs.com/demo-newsletter-editor.html)
- [grapesjs-mjml](https://github.com/GrapesJS/mjml) - [使用 MJML 的邮件通讯构建器演示](https://grapesjs.com/demo-mjml.html)

您可以直接使用这些预设作为您编辑器的起点，只需按照它们仓库中的说明操作，即可快速启动您的构建器。

## 下载

最新版本: [![npm](https://img.shields.io/npm/v/grapesjs.svg?colorB=e67891)](https://www.npmjs.com/package/grapesjs)

您可以通过以下任一来源下载 GrapesJS：

- CDN(内容分发网络)
  - unpkg
    - `https://unpkg.com/grapesjs`
    - `https://unpkg.com/grapesjs/dist/css/grapes.min.css`
  - cdnjs
    - `https://cdnjs.cloudflare.com/ajax/libs/grapesjs/0.12.17/grapes.min.js`
    - `https://cdnjs.cloudflare.com/ajax/libs/grapesjs/0.12.17/css/grapes.min.css`
- npm
  - `npm i grapesjs`
- git
  - `git clone https://github.com/GrapesJS/grapesjs.git`

## 更新日志(Changelog)

我们通过 [Github Releases](https://github.com/GrapesJS/grapesjs/releases) 来追踪库的变更历史。