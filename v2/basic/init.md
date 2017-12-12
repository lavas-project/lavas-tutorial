# 基本功能

Lavas 提供了一个命令行工具，帮助开发者快速初始化项目。具体的开发环境准备和初始项目命令步骤可以参见 Codelab “开发第一个 Lavas 应用”的[准备环境](/codelab/get-started/prepare)部分。

在执行了 `lavas init` 并输入对应信息之后，我们可以看到一个项目被自动初始化生成了。关于项目如何运行及效果等等在 Codelab 中已有介绍，这里不做赘述。这里主要向大家介绍一些 Lavas 提供的基本功能，作为 Codelab 中未涉及部分的补充。

## 初始目录结构

通过命令初始化项目完成之后，我们应该能够看到如下的文件结构：

```
lavas-project
├── assets/
├── components/
├── core/
├── entries/
├── middlewares/
├── node_modules/
├── pages/
├── static/
├── store/
├── lavas.config.js
├── server.dev.js, server.prod.js
├── .babelrc, .editorconfig, .fecsignore, .fecsrc, .gitignore
└── LINCENSE, package.json, README.md
```

### assets, static

把这两个目录放在一起，是因为这两个目录都是存放外部静态资源的，例如 [iconfont](http://www.iconfont.cn/)(用字体实现矢量小图标的库), icons(用于添加到手机桌面时使用的各种尺寸的图标), manifest.json(同样用于添加到桌面时使用)等。

这些静态资源在构建时会一并被放入生成目录中(`/dist`)，但两者也有差别：

* `/assets` 里的内容会被 webpack 构建到生成目录的文件中，不再会单独以文件形式存在。因此 iconfont 放置在 `/assets` 中

* `/static` 里的内容会被原样复制到生成目录中，会以独立的文件形式存在。因此 PWA 用到的 manifest.json 和一系列图标等都放置在 `/static` 中。

### components

`/components` 存放 Vue 的组件，供其他页面复用。在 Lavas 中，初始状态下提供了三个组件，均在一些页面框架中使用，因此会作用于整个项目的所有页面。

1. `OfflineToast.vue` 在 `/entries/main/App.vue` 中被引用，用于在用户网络离线/恢复时提示用户

2. `UpdateToast.vue` 也在 `/entries/main/App.vue` 中被引用，用于 Service Worker 更新时提示用户

3. `ProgressBar.vue` 在 `/entries/main/entry-client.js` 中被引用，在页面切换时在顶部展示加载的进度条

如果开发者有其他多个页面需要复用的组件，也可以放在 `/components` 目录中。

### core

`/core` 目录中存放一些散落但必须的文件，初始状态下有四个文件：

1. entry-server.js

    熟悉 [webpack](https://webpack.js.org/) 和 [Vue SSR](https://ssr.vuejs.org/zh/) 的开发者应该了解我们需要 `entry-client.js` 和 `entry-server.js` 两个文件。Lavas 在 Vue SSR 的基础上增加了入口 (entry)。 `entry-client.js` 是每个入口独立拥有的，因此位于 `/entries/xxx/entry-client.js`；而服务器因为只有一个，所以 `entry-server.js` 只有一份，不应该放置在每个入口中，所以单独地放在外面的 `/core` 目录中。关于 Lavas 入口 (entry) 的更多信息可以参考 [文档的 entry 部分](/v2/advanced/entry)

2. middleware.js

    Lavas 为开发者提供了中间件(middleware)功能，开发者可以在 `/middlewares` 目录中编写自定义的中间件，每次网络请求都会通过中间件，从而对网络请求(如头信息、参数等等)进行判断或修改。有关中间件的详情可以参考 [文档的中间件部分](/v2/advanced/middleware)

3. service-worker.js

    Lavas 内部使用 [Workbox](https://github.com/GoogleChrome/workbox) 进行 Service Worker 的管理和生成。`/core/service-worker.js` 用作 Workbox 生成 `service-worker.js` 的模板。这部分将在 [文档的 Service Worker 部分](/v2/advanced/service-worker)详细阐述

4. store.js

    Lavas 是基于 Vue 的解决方案，其中当然也包括了用以处理数据的 Vuex。 `/core/store.js` 用于将位于 `/store` 目录中的__所有__ js 文件当做 Vuex 模块 (module) __自动__引入，开发者无需再进行额外配置。
