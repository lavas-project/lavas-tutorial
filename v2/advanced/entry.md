# 入口 (entry)

入口 (entry) 可以理解为多个页面 (page) 的集合，一个入口内部可以共享一些配置，包括渲染方式(是否服务端渲染)， 路由方面 (`baseUrl`, `mode`)，和整个页面的外部框架 (index.html, App.vue) 等等。不同的入口之间也可以共享组件 (`/components/`) 和服务器配置(如果有的话)等。

让我们看几个比较常见的入口使用场景来深入了解：

* 一个站点既提供 PC 服务又提供移动端服务，并且页面并非响应式布局(即 PC 和移动端使用完全不同的两套页面)，那么就可以创建两个入口来分别管理。

* 一个站点的两个模块在渲染模式 (是否服务端渲染)， 路由规则 (hash 或者 history) 或者外观样式、配色等不尽相同，则可以将这两个模块处理为两个入口。例如电商网站的母婴品类和数码3C品类。

这里主要将介绍：

1. 入口的目录结构和每个文件的作用
2. 入口的配置方法和每个配置项的含义
3. 如何添加/删除入口

## 入口的目录结构

通过 `lavas init` 的项目初始状态包含一个默认入口 `main`。我们先观察一下这个入口的目录结构：

```
entries
└── main
    ├── app.js
    ├── App.vue
    ├── entry-client.js
    ├── index.html.tmpl
    └── Skeleton.vue
```

### app.js

创建整个入口的 Vue 实例，引用入口的所有内容(路由规则，Vue 组件，Vuex store等)，被 `entry-server.js` 和 `entry-client.js` 使用。了解 Vue 的开发者对这个文件应该相当熟悉。

### App.vue

整个入口的布局框架，会应用于入口内的所有页面。它的本身也是个 Vue 组件，可以通过修改这个组件实现不同入口的不同样式。从 template 中包含的 `<router-view>` 标签可以看出，这里使用 vue-router 让入口内所有页面来填充内容并管理路由。关于 vue-router 的知识这里就不作过多介绍了。

### entry-client.js

Vue SSR 要求我们建立 `entry-client.js` 和 `entry-server.js` 分别作为 webpack 的两端入口，两者各存在一份。但因为 Lavas 支持了多入口，为了满足开发者在不同入口中使用不同的前端类库且不互相影响(如 PC 端不需要引用 FastClick， 移动端不需要引用老版本浏览器兼容类库等)，`entry-client.js` 需要多份并存在于每个入口下。

排除多份 `entry-client.js` 这个差异之外，`entry-client.js` 和 `entry-server.js` 的基本逻辑和 Vue SSR 是相同的。这里 `entry-client.js` 的工作是

* 和服务端同步状态 (\_\_INITIAL\_STATE\_\_)
* 注册并调用中间件
* 在客户端调用 `asyncData()` 为每个页面获取数据
* 注册其他路由事件 (如 `router.beforeEach` 等)

### index.html.tmpl

服务端渲染模式下，我们需要新建一个 `index.template.html` 作为页面基本框架。其中最重要的 `<!--vue-ssr-outlet-->` 将作为 Vue SSR 的 HTML 注入标记。在客户端模式下，我们也需要提供一个类似的模板文件。虽然不需要 HTML 注入点，但也需要一个 `<div id="app">` 从而供 Vue 的 `app.$mount('#app')` 使用。一般来说，这就是这两个模板的唯一差别，因此 Lavas 在它们的基础上进行优化，开发者只需要编写一份模板，由 Lavas 分别生成两份供不同的模式使用，这就是 `index.html.tmpl`

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
    输出配置在入口中的 baseUrl。在开发者需要使用相对路径引用项目内的资源时，添加 `<%= baseUrl%>some/path` 可以和入口的路由配置同步。注意 `baseUrl` 一般最后都已经包含了 `/`。

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

### Skeleton.vue

Lavas 在知乎专栏上发表过一篇文章：[为 Vue 项目添加骨架屏](https://zhuanlan.zhihu.com/p/28465598)，详细讲述了什么是 skeleton 及其优势。这里的 `Skeleton.vue` 就是用作渲染骨架屏的 Vue 组件。开发者只需要将符合自身站点显示风格的图片替换进去，即可在 MPA/SPA 等纯前端渲染模式下看到骨架屏，提升用户体验。

骨架屏在服务端渲染的情况下__并不生效__。为了解决这个问题，Lavas 探索并实现了另外一种实现方案，这部分将在 [App Shell](/v2/advanced/appshell) 进行介绍。

## 入口的配置项

除了 `/entries/` 目录中入口本身的内容之外，保证入口正常运作的还有入口的配置项，这部分位于 `/lavas.config.js` 中的 `entry` 部分。同样拿初始化的项目举例，我们可以看到这样的初始配置。

```javascript
// ...
entry: [
    {
        name: 'main',
        ssr: true,
        mode: 'history',
        base: '/',
        routes: /^.*$/,
        pageTransition: {
            type: 'fade',
            transitionClass: 'fade'
        }
    }
],
// ...
```

### name

指定入口的名称，必须和 `/entries/` 中的入口目录名称__保持一致__，包括大小写。因此推荐全部小写。

### routes

`routes` 可以说是入口配置项中最重要的一项。在将上一节讲到的入口内容添加完成后，我们还需要通过配置告知 Lavas 哪些流量应该被导流到这个入口，这就是 `routes` 的作用。

`routes` 的类型可以是字符串，或者正则表达式，或者这两种类型的数组。如果是字符串，Lavas 会转化成为正则表达式进行匹配；如果是数组，则匹配其中任意一项均视为匹配。另外，因为 `/lavas.config.js` 的 entry 是一个数组，因此 Lavas 会按序匹配，越往前的越会被优先匹配。

如例子中的 `/^.*$/` 可以匹配任何 URL，因此作为默认入口是合理的。如果某个 URL 无法被任何入口匹配，则会抛出一个错误。所以使用例子中的正则表达式匹配所有 URL 并放置在 entry 数组的最后是比较稳妥的方式。

### ssr

指定是否采用服务端渲染。

如果为 `false`，则使用常规的 Vue 客户端渲染方式。这时还可以区分为 SPA (单页应用)和 MPA (多页应用)两种模式，差别在于存在单个还是多个入口。

如果为 `true`，则采用 [Vue SSR](https://ssr.vuejs.org/) 进行服务端渲染。在这种模式下，首屏请求的内容会由服务器渲染后直接给出，而不是像 SPA 或者 MPA 那样，服务器给出页面框架，由前端填充内容。因此从 SEO 角度来说有比较明显的提升，当然复出的代价就是首屏渲染速度较客户端渲染慢一些。不过这个差别只存在于首屏请求，在后续的页面跳转依然采用客户端渲染。

SSR 的首屏渲染性能问题可以通过 Service Worker 的 App Shell 模型大幅改善，如有兴趣可以参考[这里](/v2/advanced/appshell)。

### mode, base

这两个其实都是 vue-router 的配置项，分别用来定义路由模式和路由基础路径。

`mode` 可选值有 `'hash'` 和 `'history'` (`'abstract'` 只用作服务端使用，这里并没有使用的必要)。注意的是在 SSR 模式下__不支持__ `hash`。



更多信息可以参考 vue-router 的文档中 [mode](https://router.vuejs.org/zh-cn/api/options.html#mode) 和 [base](https://router.vuejs.org/zh-cn/api/options.html#base) 部分。

### pageTransition

## 创建入口

假设我们要创建一个入口 `user` 用以负责所有用户信息相关的工作(登录/注册/查看资料/修改资料等)，对应的模块 URL 为 `/user/*`。我们可以通过 Lavas 内置命令进行入口创建。

```bash
lavas addEntry user
```

![lavas-addEntry](http://boscdn.bpc.baidu.com/assets/lavas/codelab/lavas-addEntry.png)

Lavas 帮助我们快速在 `/entries/` 目录下创建了 user 目录，并创建了一整套入口需要的文件。

下一步是在 `/lavas.config.js` 的 entry 配置数组增加导流配置。这个数组的作用是根据用户请求(通常为 URL ) 来判断哪些请求属于哪个入口，因此我们需要在这里建立新入口的 URL 规则，从而将请求导流到新的入口上。

```javascript
// ...
entry: [
    {
        name: 'user',
        ssr: true,
        mode: 'history',
        base: '/',
        routes: /^\/user/,
        pageTransition: {
            type: 'fade',
            transitionClass: 'fade'
        }
    }
    // other entries
],
// ...
```

