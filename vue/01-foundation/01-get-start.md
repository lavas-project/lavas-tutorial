# 开始探索 PWA 工程

## 什么是 PWA

PWA (Progressive Web Apps) 是一种 Web App 新模型，并不是具体指某一种前沿的技术或者某一个单一的知识点，我们从英文缩写来看就能看出来，这是一个渐进式的 Web App，是通过一系列新的 Web 特性，配合优秀的 UI 交互设计，逐步的增强 Web App 的用户体验。 

用户的手机现在几乎被各种大大小小形形色色的 App 给攻占了，手机的容量是有上限的，用户的时间成本也是有上限的，我们如何让 App 做到用户需要就能立马得到，用户不需要不占用手机资源呢？显然 Native App 是根本做不到这点的，用户能做的只会在抱怨和留舍纠结中一遍一遍的重复着安装和卸载。而另一方面 Native App 没法做到平台兼容，对于企业和开发者来说相对于 Web App 的平台兼容性以及可维护行来说 Native App 的开发运维成本太高。

PWA 工程的解决方案中借助了 service worker 的离线存储能力，消息推送能力以及系统的添加桌面能力，从而形成一个完善的 Web App 解决方案，帮助我们在 Web 端低成本的开发和维护一个逐步类 Native App 化的 Web App。

我们看一下 [Google 对 PWA 全面介绍](https://developers.google.com/web/progressive-web-apps)。


## 什么站点适合改造成 PWA

PWA 的理念是通过 Web 的新特性，将我们的 Web App 改造升级，逐步替代 Native App 从而解放用户的设备和提升用户体验。
原则上除了对系统强依赖的 App, 以及游戏类的 App 等, 所有的 Native App 都可以改造成 PWA 应用。


## PWA 的效果

PWA 工程构建出的 App 是一个 Web App，我们需要做的是使这样的一个 Web App 在体验上和 Native App 保持一致，并且不占用用户设备的存储空间。通常我们需要确保我们的工程完成了如下工作。

- https 环境部署。
- 响应式设计，一次部署，可以在移动设备和 PC 设备上运行。
- 在不同浏览器下可正常访问。
- 浏览器离线和弱网环境可极速访问。
- 可以把 App icon 入口添加到桌面。
- 点击 icon 入口有类似 Native App 的动画效果。
- 灵活的热更新机制
- ...

对于 PWA 的效果， Google 给出了 PWA 验证的系列 [checkList](https://developers.google.com/web/progressive-web-apps/checklist)， 我们可以参考比对，检验自己的 PWA 工程。

## 离线缓存

传统的 Web App 在没有网络或者弱网的情况下就毫无用处了，这个是没有办法的，因为传统 Web App 的一切包括浏览器渲染的内容以及所依赖的一切静态资源都是基于 http 请求的，所以当网络情况恶劣的情况下，自然请求缓慢甚至失效，导致用户体验大打折扣。

Web 发展进程中，我们有过很多的方法来优化因为 http 请求的原因导致的用户体验问题，这些都是治标不治本的行为，现在 HTML5 新的 [serviceWorker API](https://pwa.baidu.com/doc/offline-and-cache-loading/01-service-worker-introduction), 提供的离线缓存能力，颠覆了传统 Web App 的运作形式。

我们可以将一切 Web App 运行时必要的资源持久本地缓存起来，当无网或者弱网的时候，直接从缓存中取到资源，进行正常的 Web App 的渲染和执行。这样，在离线这块，Web App 是可以和 Native App 所媲美的。


## SPA (Single Page Apps)

Native App 有一大优势是流畅，交互流畅和切换的流畅，然而传统的 Web App 在这方面是劣势的缘由主要是因为我们的切换场景的做法仅仅是跳转刷新新页面，这样的交互会使得整个 Web App 用户操作极其不流畅并且浪费了多余的网络流量。

为了实现流畅的 Web App 页面切换以及缩短页面打开和跳转的等待时长，业界在过去几年间进行了不断的探索，从 Ajax 开始，越来越多的单页富应用 (SPA)问世，并且在实现和体验上有着非常高效的解决方案，考虑到异步请求的可缓存性以及用户体验的流畅性，我们在 PWA 工程的实践中首选的工程架构模式是 SPA 模式

SPA 架构中我们聚焦的是异步请求，数据以及交互之间的关系。对于开发者来说，除了初始的 App 框架 Layout, Web App 就是由用户的一个个交互动作处罚的异步请求的数据驱动渲染成各个页面的各个模块的，然后展现给用户，用户进而进行新的交互操作。

SPA 架构能够大大提升 Web App 在交互上的流畅性和可交互性，并为交互的多种可能性提供了实践的基础。在这方面，由于浏览器实践的限制以及系统层面的原因，Web App 的流畅性虽然暂时没有 Native App 那么流畅，但是 Native App 的交互动画效果，都可以通过最新的 Web 技术在 Web App 上实现出来。

我们可以深入[了解 SPA 相关内容](http://blog.angular-university.io/why-a-single-page-application-what-are-the-benefits-what-is-a-spa)。



## 基于 Vue 架构的 PWA 工程

[Vue](https://cn.vuejs.org/v2/guide) 是一套构建用户界面的渐进式框架，与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue 的核心库只关注视图层，它不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与单文件组件和 Vue 生态系统支持的库结合使用时，Vue 也完全能够为复杂的单页应用程序提供驱动。 

PWA 工程是一套渐进式的解决方案，我们通常需要架构在 SPA 的基础上，而 Vue 的渐进式设计理念很适合我们构建一套 SPA 的工程。所以我们本教程主要是描述如何开发和维护一个基于 Vue 架构的 PWA 工程。

在开始 PWA 工程开发之前，我们要确保我们对 Vue 有了较为全面的了解，Vue 的文档建设非常全面，我们可以去 [Vue 官网](https://cn.vuejs.org)进行学习。


## App Shell

App Shell 架构是构建 PWA 工程的一种方式，这种应用能可靠且即时地加载到您的用户屏幕上，与本地应用相似。

App Shell 是支持用户界面所需的最小的 HTML、CSS 和 JavaScript，如果离线缓存，可确保在用户重复访问时提供即时、可靠的良好性能。这意味着并不是每次用户访问时都要从网络加载 App Shell。 只需要从网络中加载必要的内容。

对于使用包含大量 JavaScript 的架构的 SPA 来说，App Shell 是一种常用方法。这种方法依赖渐进式缓存 Shell（使用 Service Worker 线程）让应用运行。接下来，为使用 JavaScript 的每个页面加载动态内容。App Shell 非常适合用于在没有网络的情况下将一些初始 HTML 快速加载到屏幕上。

App Shell 通常是由服务端渲染，并不是在前端由数据驱动渲染的，所以在 SEO 方面也有着重要的作用。

App Shell 在 PWA 工程化中扮演重要角色，Google 对 [App Shell 也有详细的描述](https://developers.google.com/web/fundamentals/architecture/app-shell)。


## 工程化

本教程将一步一步的领着你进入 PWA 工程化之旅，我们通过一系列的解决方案帮助你完成一个完善的 PWA 应用。

基本了解了 PWA 相关背景和介绍之后，可以开始阅读后面的工程化具体实践教程了，我们开始吧～
