# core 目录

如果使用 `lavas init` 命令初始化项目，core 目录有如下文件：

* app.js
* App.vue
* entry-client.js
* entry-server.js
* index.html.tmpl
* service-worker.js
* Skeleton.vue

和其他目录相比，core 目录相对较为松散，这些文件的作用各不相同，下面分别进行介绍。

## app.js

创建整个项目的 Vue 实例，引用其他 Vue 相关的内容(路由规则，Vue 组件，Vuex store等)，被 `entry-server.js` 和 `entry-client.js` 使用。了解 Vue 的开发者对这个文件应该相当熟悉。

## App.vue

整个项目的布局框架，会应用于项目内的所有页面。从 template 中包含的 `<router-view>` 标签可以看出，这里使用 vue-router 管理路由。关于 vue-router 的知识这里就不作过多介绍了。

## entry-client.js

Vue SSR 要求我们建立 `entry-client.js` 和 `entry-server.js` 分别作为 webpack 的两端入口，两者各存在一份。这里 `entry-client.js` 的工作包括：

* 和服务端同步状态 (\_\_INITIAL\_STATE\_\_)
* 注册并调用中间件
* 在客户端调用 `asyncData()` 为每个页面获取数据
* 注册其他路由事件 (如 `router.beforeEach` 等)

开发者可以在 [Vue SSR 官方文档](https://ssr.vuejs.org/zh/structure.html) 中找到相关的信息。

## entry-server.js

如上所述，这里的 `entry-server.js` 作为 webpack 编译服务端的入口。它的作用包括：

* 执行服务端路由匹配和数据预期逻辑
* 将必须的信息挂载在 `context` 上，供 SSR 情况下 Vue 组件使用，如 `store`, `route`, `meta` 等等
* 注册并执行中间件 (middleware)

开发者可以在 [Vue SSR 官方文档](https://ssr.vuejs.org/zh/structure.html) 中找到相关的信息。

## index.html.tmpl

在服务器端渲染模式下，Vue SSR 要求开发者提供一份 HTML 模板文件作为页面基本框架。其中最重要的 `<!--vue-ssr-outlet-->` 将作为 HTML 注入标记。在浏览器端渲染模式下，我们也需要提供一个类似的模板文件。虽然不需要 HTML 注入点，但也需要一个 `<div id="app">` 从而在 Vue 的 `app.$mount('#app')` 时使用。

一般来说，这就是这两个模板的唯一差别，因此 Lavas 在它们的基础上进行优化，开发者只需要编写一份模板，由 Lavas 分别生成两份供不同的模式使用，这就是 `index.html.tmpl`

一个简单的示例模板如下：

```
<html>
    <head>
        <%= renderMeta() %>
        <%= renderManifest() %>
    </head>
    <body>
        <%= renderEntry() %>
    </body>
</html>
```

Lavas 提供的内置方法有这么几个：

* `renderMeta()`

    使用 [vue-meta](https://github.com/declandewet/vue-meta) 进行 meta 信息的渲染，包括 title, titleTemplate 等等，详见 vue-meta 的使用方法。

* `renderManifest()`

    给浏览器提供 `manifest.json` 以激活“添加到手机桌面”的功能。这同时要求开发者配置合法的 `/static/manifest.json`。关于 JSON 的配置可以参考 [Codelab](/codelab/get-started/manifest)。

* `renderEntry()`

    渲染 HTML 入口。上面提过，对于服务端渲染来说，就是 `<!--vue-ssr-outlet-->` 这个标签，而对于客户端来说则是 `<div id="app"></div>`。 一般情况这个方法__必须__要有，否则 Vue 将会因为找不到入口而无法渲染 HTML。

* `baseUrl`

    输出配置在 `lavas.config.js` 中 `router` 的 `base`。在开发者需要使用相对路径引用项目内的资源时，添加 `<%= baseUrl%>some/path` 可以保证路径一致。注意 `baseUrl` 一般最后都已经包含了 `/`。

* `useCustomOnly()`

    如果开发者希望使用自定义的模板(例如某些特别的框架，不需要生成完整的 HTML 结构)，可以使用这个方法来屏蔽所有 Lavas 做的预处理，直接使用用户编写的内容。

除了调用以上几个方法，在其他任何位置插入 HTML 标签__都是允许__的，从而最大限度的为开发者提供灵活性。因此如下模板也是合法的：

```
<html>
    <head>
        <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, minimal-ui">
        <%= renderMeta() %>
        <link rel="shortcut icon" href="<%= baseUrl %>static/img/icons/favicon.ico">
        <%= renderManifest() %>
    </head>
    <body>
        <div class="main-entry">
            <%= renderEntry() %>
        </div>
    </body>
</html>
```

### index.html.tmpl 和 App.vue

开发者可能注意到了，`index.html.tmpl` 和 `App.vue` 都可以充当布局框架使用，在这方面他们有什么区别呢？

从纯粹技术实现上来说，`index.html.tmpl` 可以处理整个 HTML，即包括 `head` 和 `body`；而 `App.vue` 只是 `body` 的一部分，控制范围小了一些。因此如果需要在 `head` 中进行修改，就必须使用 `index.html.tmpl`，而在 `body` 中的话，两者都能实现。

从代码风格和编程习惯来看，如果是在 `body` 中的修改(最常见的就是增加顶栏、侧边栏、菜单栏或者修改显示样式等)，一般也倾向于放在 `App.vue` 中，毕竟是个 vue 文件，可以使用的方法，变量，判断等都更为丰富。


## service-worker.js

Lavas 内部使用 [Workbox](https://github.com/GoogleChrome/workbox) 进行 Service Worker 的管理和生成。`/core/service-worker.js` 用作 Workbox 生成 `service-worker.js` 的模板。这部分将在 [文档的 Service Worker 部分](/guide/v2/advanced/service-worker)详细阐述

## Skeleton.vue

`Skeleton.vue` 是用作渲染骨架屏的 Vue 组件。Lavas 在知乎专栏上发表过一篇文章：[为 Vue 项目添加骨架屏](https://zhuanlan.zhihu.com/p/28465598)，详细讲述了什么是骨架屏及其优势。这里开发者只需要将符合自身站点显示风格的图片替换进去，即可在浏览器端渲染模式 (SPA) 下看到骨架屏，提升用户体验。

骨架屏在服务端渲染的情况下__并不生效__。为了解决这个问题，Lavas 探索并实现了另外一种实现方案，这部分将在 [Skeleton 和 App Shell模型](/guide/v2/advanced/appshell) 中进行介绍。
