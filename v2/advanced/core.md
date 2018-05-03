# core 目录

如果使用 `lavas init` 命令初始化项目，core 目录有如下文件：

* app.js
* App.vue
* entry-client.js
* entry-server.js
* spa.html.tmpl
* ssr.html.tmpl
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

## spa.html.tmpl & ssr.html.tmpl

*在旧版中，两者统一为 `index.html.tmpl`，通过 `if (ssr)` 进行判断*

在服务器端渲染模式下，Vue SSR 要求开发者提供一份 HTML 模板文件作为页面基本框架。其中最重要的 `<!--vue-ssr-outlet-->` 将作为 HTML 注入标记。在浏览器端渲染模式下，我们也需要提供一个类似的模板文件。虽然不需要 HTML 注入点，但也需要一个 `<div id="app">` 从而在 Vue 的 `app.$mount('#app')` 时使用。

一般情况下，在项目调研阶段可能会在 SPA 和 SSR 两种渲染模式中互相切换，在进入大规模开发时只会使用其中的一种渲染模式。为了防止另一种渲染模式的代码依然留存在开发者的代码库中，Lavas 将两种模式的模板进行分割，分别命名为 `spa.html.tmpl` 和 `ssr.html.tmpl`。因此为了项目精简考虑，__如果您的项目确定使用某一种渲染方式，您可以删除另外一种渲染方式的模板__。

在 SSR 模式下，Lavas 还支持开发者传入变量并使用。具体方式如下：

1. 在 `core/entry-server.js` 中，将需要使用的变量挂载到 `context` 对象上。如

    ```javascript
    context.author = {name: 'wangyisheng'};
    ```

2. 在 `ssr.html.tmpl` 中，使用 `{{}}` 的样式使用这个变量。如

    ```
    <div class="author">{{ author.name }}</div>
    <!-- 或者 -->
    <input value="{{ author.name }}"></input>
    ```

在 SPA 模式下，这种方式__并不能奏效__。原因也很简单，我们可以从 SPA 和 SSR 的原理上进行考虑。SSR 模式下，由服务器(中间件)获取请求，并把创建的 Vue app 和 context 交给 `renderToString` 方法进行渲染，因此可以进行模板变量的注入和替换；而 SPA 模式下，服务器只负责使用 webpack 进行构建，构建时并不真正运行代码 (entry-client.js)，因此无法获取变量，也就无法替换了。

### ssr.html.tmpl (spa.html.tmpl) 和 App.vue

开发者可能注意到了，`spa.html.tmpl` (`ssr.html.tmpl`) 和 `App.vue` 都可以充当布局框架使用，在这方面他们有什么区别呢？

从纯粹技术实现上来说，模板文件可以处理整个 HTML，即包括 `head` 和 `body`；而 `App.vue` 只是 `body` 的一部分，控制范围小了一些。因此如果需要在 `head` 中进行修改，就必须使用模板文件，而在 `body` 中的话，两者都能实现。

从代码风格和编程习惯来看，如果是在 `body` 中的修改(最常见的就是增加顶栏、侧边栏、菜单栏或者修改显示样式等)，一般也倾向于放在 `App.vue` 中，毕竟是个 vue 文件，可以使用的方法，变量，判断等都更为丰富。


## service-worker.js

Lavas 内部使用 [Workbox](https://github.com/GoogleChrome/workbox) 进行 Service Worker 的管理和生成。`/core/service-worker.js` 用作 Workbox 生成 `service-worker.js` 的模板。这部分将在 [文档的 Service Worker 部分](/guide/v2/advanced/service-worker)详细阐述

## Skeleton.vue

`Skeleton.vue` 是用作渲染骨架屏的 Vue 组件。Lavas 在知乎专栏上发表过一篇文章：[为 Vue 项目添加骨架屏](https://zhuanlan.zhihu.com/p/28465598)，详细讲述了什么是骨架屏及其优势。这里开发者只需要将符合自身站点显示风格的图片替换进去，即可在浏览器端渲染模式 (SPA) 下看到骨架屏，提升用户体验。

骨架屏在服务端渲染的情况下__并不生效__。为了解决这个问题，Lavas 探索并实现了另外一种实现方案，这部分将在 [Skeleton 和 App Shell模型](/guide/v2/advanced/appshell) 中进行介绍。
