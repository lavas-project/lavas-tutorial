# 简介

Lavas 是一套基于 Vue 的 PWA 解决方案，能够帮助开发者快速搭建 PWA 应用，解决接入 PWA 的各种问题，对提升用户体验，用户留存率等有明显提升，且开发者无须过多的关注 PWA 开发本身。

如果您对 PWA 的概念还不熟悉，可以参考 Lavas 官网中关于 PWA 的[介绍](https://lavas.baidu.com/doc)。简而言之，PWA 的目标是让移动端的 H5 站点拥有可以媲美本地 APP 的体验，包括离线可访问，添加桌面图标以快速启动等等特性。加之移动站点无须频繁安装升级的优势，进而挑战现有 APP 的用户习惯，建立 WEB 新生态。

## 学习本教程前您应该掌握

* HTML, CSS(less或stylus), JavaScript 等前端编程基础
* ES6/7 部分常用语法，如 Promise, import/export 等
* Vue, Vuex, Vue-router 等基本的开发知识 ([教程](https://cn.vuejs.org/v2/guide/)、[API 文档](https://cn.vuejs.org/v2/api/) 和 [编程风格指南](https://cn.vuejs.org/v2/style-guide/))
* Webpack, Node.js 等基本知识 （__推荐__）

Lavas 后续的开发计划也已经将支持其他主流前端框架 (如 react) 纳入其中，如果您持续关注我们([Github](https://github.com/lavas-project/lavas))，给我们多提意见甚至代码，我们将会非常感谢！

## Lavas 能做什么

Lavas 解决方案能够帮助开发者完成：

* 最基本的移动站点建设，包括 Vue, Vuex, Vue-router, webpack 等常用且成套的技术提供支持

* 允许站点添加至手机桌面，再次打开时全屏运行，摆脱浏览器的固定显示框架(地址栏，菜单栏等)

* 强化缓存，允许站点在弱网甚至离线的情况下继续显示内容

* 支持消息推送，帮助站长主动推送用户感兴趣的信息，提升业务指标

* 支持服务端渲染(SSR)，对搜索引擎更加友好

* 支持[App Shell 模型](https://lavas.baidu.com/doc/architecture/the-app-shell-model)，在正常情况下提升加载性能，在异常情况下优化错误显示。

总结起来，Lavas 除了能帮助开发者完成 Vue 能做的(搭建基本站点)之外，通过一些配置还能够快速赋予站点 PWA 的特性，且不需要开发者过多的关心 PWA 的详细开发技术和细节。

我们可以粗略的理解为： `Lavas = Vue + PWA`
