# 探索 Lavas

## Lavas 是什么

Lavas 是一个基于 Vue 的 PWA 解决方案。我们将 PWA 的工程实践总结成多种应用框架模板，帮助开发者快速搭建 PWA 应用。


## Lavas 在做了什么

简单来说，站点 PWA 化需要做什么，Lavas 就做了什么。它提供基于 Vue 框架的完整解决方案，让开发者能更多的关心业务需求本身。

站点 PWA 化，就是要使站点具有类似原生应用的功能和体验，如站点可添加主屏幕、全屏方式运行、支持离线缓存、消息推送等。所以要实现站点的 PWA 改造，下面几个工作就必不可少：

1. 为了让站点能像原生应用那样安装到主屏幕，我们需要准备“图标”这些静态资源，需要准备一个清单文件 [manifest.json](https://lavas.baidu.com/doc/engage-retain-users/add-to-home-screen/01-introduction) 去告知浏览器使用哪些图标，显示哪些应用名称等等；

2. 让站点具有更好的离线体验。PWA 提供了更好的缓存 API （详见[web 存储](https://lavas.baidu.com/doc/offline-and-cache-loading/web-storage/01-overview)）和缓存管理方式 [Service Worker](https://lavas.baidu.com/doc/offline-and-cache-loading/service-worker/01-service-worker-introduction)，而具体的缓存策略仍然需要开发者根据项目的实际需要进行开发；

3. 同样是为了让站点具有更好的离线体验，除了要在缓存策略上下功夫，站点 UI 设计上也需要遵循一定的规范（如[App Shell 模型](https://lavas.baidu.com/doc/architecture/the-app-shell-model)和[离线 UX 注意事项](https://lavas.baidu.com/doc/offline-and-cache-loading/offline-ux-considerations)），以至于站点在页面切换、内容加载、加载出错、弱网断网等等情况下不会给用户显示个大白屏。

上述这些工作，我们希望可以通过 Lavas 工具来解决，而开发者只需要做一些业务配置，即可实现，大大节约开发成本。



## 快速掌握 Lavas

随着下面的指引，可以帮助您快速的掌握 Lavas，废话不多说，let's go!

教程文档分为基础和进阶， 基础部分可以帮助大家快速搭建项目并跑通，让您对项目有一个初步的了解和体验。进阶部分可以为大家剖析一些项目中必要技术点的实现，包括配置使用方法、使用时注意的问题、如何避免一些曾经踩过的坑等等，为使用者开发大型项目时提供更好的技术支持。

**基础教程**

1、如果您对 PWA 本身还不是很了解，可以通过 [探索 PWA ](https://lavas.baidu.com/guide/vue/doc/vue/01-foundation/01-get-start)来快速了解其一些特性，优势等，这会让您对项目的目标更加清晰。

2、参考[快速开始 PWA 工程](https://lavas.baidu.com/guide/vue/doc/vue/01-foundation/02-quick-tour-by-cli)，使用 Lavas 工具快速导出一个 PWA 工程，浏览项目结构，运行浏览模板效果。

- Lavas 工具中提供了多种应用模板框架，既包括最轻量的 basic 模板，appshell 模板，及多页应用的 MPA 模板，大家按需参考使用
- 初始化项目模板预设了 manifest.json 文件和 Service Worker 相关插件，均有默认设置
- 初始化的框架结构，遵循 UI 开发规范，可引导用户遵循一定的开发规则，更高效地进行项目管理
- 参考其他几节来快速掌握简单的开发、调试及部署技巧。


**进阶教程**

1、[添加到桌面功能](https://lavas.baidu.com/doc/engage-retain-users/add-to-home-screen/01-introduction)： 配置桌面的图标、文案、打开方式、主题色等，目前是在 `static/manifest.json` 目录下默认配置，使用者可以按需管理更换。

2、Service Worker：该部分主要是需了解离线资源缓存管理与更新，项目默认缓存所有静态资源，如果您想缓存指定内容，或配置部分动态缓存的内容及相关问题，均可参考进阶教程中[维护 service-worker.js 文件](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/05-service-worker-maintenance) 和 [Service Worker 与页面通信](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/06-service-worker-postMessage)两部分内容来寻找解决方案。

3、App Shell：简单说就是第一次渲染个壳、等异步数据来了再填充，避免用户长时间看到白屏，这部分可以在[App Shell 调整及扩展](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/01-define-app-shell) 中查看相关介绍。但是 App Shell 首先渲染出来后，Shell 内部依旧是白的，如果您想更深入去解决这个问题，可以参考[ skeleton 介绍](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/07-skeleton)实现在项目渲染过程中，先看到页面框架，然后用真实内容替换，提升用户体验。


3、在除 basic 外的其他模板中，提供了一些比较实用的功能，如[项目主题色修改](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/04-how-to-change-theme)、[项目中 icon 图标使用](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/03-how-to-use-icon)等。


4、进阶部分还对[页面切换动画](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/09-animation)、[https 部署](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/10-https-deploy)、[Material UI 使用](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/02-material-ui)等做了详细介绍。


使用 Lavas 让你的网页可以渐进式地变成 App，欢迎大家多多适用。


