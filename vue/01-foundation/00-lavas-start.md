#  Lavas 介绍

## Lavas 是什么

Lavas 是一个基于 Vue 的 PWA 解决方案。我们将 PWA 的工程实践总结成多种应用框架模板，帮助开发者快速搭建 PWA 应用。


## Lavas 做了什么

简单来说，站点 PWA 化需要做什么，Lavas 就做了什么。它提供基于 Vue 框架站点 PWA 化的完整解决方案，让开发者能更多的关心业务需求本身。

站点 PWA 化，就是要使站点具有类似原生应用的功能和体验，如站点可添加主屏幕、全屏方式运行、支持离线缓存、消息推送等。所以要实现站点的 PWA 改造，下面几个工作就必不可少：

- 为了让站点能像原生应用那样安装到主屏幕，我们需要准备“图标”这些静态资源，需要准备一个清单文件 [manifest.json ](https://lavas.baidu.com/doc/engage-retain-users/add-to-home-screen/01-introduction) 去告知浏览器使用哪些图标，显示哪些应用名称等等；

- 让站点具有更好的离线体验。PWA 提供了更好的缓存 API （详见[web 存储](https://lavas.baidu.com/doc/offline-and-cache-loading/web-storage/01-overview)）和缓存管理方式 [Service Worker](https://lavas.baidu.com/doc/offline-and-cache-loading/service-worker/01-service-worker-introduction)，而具体的缓存策略仍然需要开发者根据项目的实际需要进行开发；

- 同样是为了让站点具有更好的离线体验，除了要在缓存策略上下功夫，站点 UI 设计上也需要遵循一定的规范（如[ App Shell 模型 ](https://lavas.baidu.com/doc/architecture/the-app-shell-model)和[ 离线 UX 注意事项 ](https://lavas.baidu.com/doc/offline-and-cache-loading/offline-ux-considerations)），以至于站点在页面切换、内容加载、加载出错、弱网断网等等情况下不会给用户显示个大白屏。

而上述这些工作，我们希望都可以通过 Lavas 来帮开发者完成，开发者只需要做一些简单的业务配置即可，从而大大节约开发维护成本。



## Lavas 教程索引

Lavas 教程文档主要分为基础和进阶两部分，也是我们建议使用者浏览的顺序。基础部分可以帮助大家快速搭建项目并跑通，让您对项目结构和效果有一个初步的了解和体验。进阶部分可以为大家剖析一些项目中必要技术点的实现，包括配置使用方法、使用时注意的问题、如何避免一些曾经踩过的坑等等，能为使用者开发大型项目时提供更好的技术支持。

#### 基础教程

- 如果您对 PWA 本身还不是很了解，可以通过 [探索 PWA ](https://lavas.baidu.com/guide/vue/doc/vue/01-foundation/01-get-start)来快速了解其一些特性，优势等，这会让您对项目的 PWA 化目标更加清晰。

- 参考[ 快速开始 PWA 工程 ](https://lavas.baidu.com/guide/vue/doc/vue/01-foundation/02-quick-tour-by-cli)，使用 Lavas 工具快速导出一个 PWA 工程，浏览项目结构，运行浏览模板效果。

    - Lavas 工具中提供了多种应用模板框架，既包括最轻量的 basic 模板，appshell 模板，也包括支持服务端渲染的 ssr 模板，支持多页应用的 mpa 模板，大家可以参考使用
    - 初始化项目模板预设了 manifest.json 文件和 Service Worker 相关插件，均有默认设置
    - 初始化的框架结构，遵循 UI 开发规范，可引导用户遵循一定的开发规则，更高效地进行项目管理

- 其他几节让您快速掌握简单的开发、调试及构建部署技巧。


#### 进阶教程

开发中，您需要关注的一些技术点，该部分的阅读会让您的开发更加得心应手：

- [ 添加到桌面功能 ](https://lavas.baidu.com/doc/engage-retain-users/add-to-home-screen/01-introduction)： 这是我们首先需要关注的一个内容，初始化项目模板中默认使用 `static/manifest.json` 中配置的添加到桌面的图标、文案、打开方式、主题色等，使用者需要在管理更换。

- Service Worker：这是 PWA 中最关键的技术，需要我们重点关注。这部分主要介绍离线资源缓存配置管理与更新，项目默认缓存所有静态资源，并提供了简单的缓存更新机制，如果您想缓存指定内容，或配置部分动态缓存的内容及离线缓存相关问题，均可参考[ 维护 service-worker.js 文件 ](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/05-service-worker-maintenance) 和 [Service Worker 与页面通信 ](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/06-service-worker-postMessage)两部分内容来寻找解决方案。

- App Shell：在解决了上面两个必须的关键问题后，您可以对页面渲染中的白屏问题做进一步的优化，App Shell 就是其中之一。简单说，它就是第一次渲染个壳、等异步数据来了再填充，避免用户长时间看到白屏。这部分可以在[ App Shell 调整及扩展 ](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/01-define-app-shell)中查看相关介绍，Lavas 工具在 basic 外的模板中增加了 App Shell 的使用示例，供您参考。

- [ skeleton 介绍 ](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/07-skeleton)：同样是为了优化白屏的问题，skeleton 的使用，能实现在页面渲染过程中，先展示整个页面内容框架，然后用真实内容替换。如果能结合 App Shell 一起使用，就能完美解决白屏的体验问题。


- 除 basic 模板外，其他模板中还提供了一些比较实用的解决方案，方便使用者开发。

    - [ 页面切换动画 ](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/09-animation)，使用者一般都需要考虑这个问题，所以这里给出了具体实现方案，供大家参考。

    - [ 项目主题色修改 ](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/04-how-to-change-theme)，开发可以通过统一配置修改站点的主题色，方便统一组件、页面的主色调。

    - [ 项目中 icon 图标使用 ](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/03-how-to-use-icon)，支持使用 material 图标、自定义 svg 图标、引入指定 fontawesome 图标等。

    - [ Material UI 使用 ](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/02-material-ui)，介绍 Material UI 及使用 ，以及开发一个自己简单组件。


Lavas 让你的网页可以渐进式地拥有 App 的体验，欢迎大家多多适用。


