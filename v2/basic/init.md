# 基本功能

## 初始目录结构

通过 `lavas init` 初始化项目完成之后，我们应该能够看到如下的文件结构：

```
lavas-project
├── assets/
├── components/
├── core/
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

1. `UpdateToast.vue` 也在 `/core/App.vue` 中被引用，用于 Service Worker 更新时提示用户

2. `ProgressBar.vue` 在 `/core/entry-client.js` 中被引用，在页面切换时在顶部展示加载的进度条

如果开发者有其他多个页面需要复用的组件，也可以放在 `/components` 目录中。

### core

`/core` 目录中存放一些散落但必须的文件，作用也相对复杂，包括 webpack entry, 全局样式框架，Vue app 启动器等等。详细可以参考 [core 目录文档](/guide/advanced/core) 进行详细查看。

### middlewares

之前在 `/core/middlewares.js` 中有提过，用户可以在这个目录编写自定义中间件，用来获取或者修改每次到达服务器的网络请求。关于中间件的具体写法及其包含的能力可以参考[文档的中间件部分](/guide/v2/advanced/middleware)。

### pages

`/pages/` 目录存放每个页面的 vue 组件。我们在开发实际站点的时候，可能大部分的工作都在这个目录中进行。每个页面组件都是一个标准的 Vue 组件，包含 `<template>`, `<script>` 和 `<style>` 三部分，开发方式也和 Vue 一模一样，这里就不再多做介绍了。

值得注意的是，`/pages/` 目录中的所有页面都会__自动生成__一条路由规则，无需用户再行配置。举例来说，在初始项目中我们看到一个 `/pages/Appshell.vue`，则 Lavas 自动生成的路由将会是 `/appshell`。更多生成规则可以参考本文下半部分的“ Lavas 自动路由生成方法” 部分。

当然如果开发者对自动生成的路由并不满意，或者有其他特殊需求需要自定义路由规则的，也可以通过 `router` 配置项进行修改，这部分将在[文档的路由部分](/guide/v2/advanced/router)进行说明。

### store

之前在 `/core/store.js` 中有提及，所有位于 `/store` 目录的 js 文件都会以 Vuex 模块 (module) 进行加载。因此开发者只需要提供一个完整的 Vuex 模块就可以在 vue 中使用它。一般来说，一个完整的 Vuex 模块需要包含以下内容：

```javascript
// someStore.js with path: /store/namespace/someStore.js

// state MUST be a function to support SSR
export const state = () => {
    // define states
};

export const mutations = {
    // define functions to change states
};

export const actions = {
    // send async requests and commit changes
};
```

这样就可以在 `/pages/` 目录中的 vue 中使用它了，例如
```javascript
// ...
computed: mapState('namespace/someStore', [
    'state1',
    'state2'
])
// ...
```

更多 store 相关的写法可以参见[文档的 store 部分](/guide/v2/advanced/store)。

### lavas.config.js

Lavas 提供了许多配置项，方便开发者进行各种自定义的灵活配置。所有的配置项都集中在 `lavas.config.js` 中，并提供一套默认配置，适用于大部分普通开发者快速上手。配置总共分为以下几个部分，您都可以在文档的进阶部分找到对应的章节进行详述：

* __build__ 构建相关，详见[这里](/guide/v2/advanced/build-config)
* __errorHandler__ 错误处理相关，详见[这里](/guide/v2/advanced/error-handler)
* __middleware__ 中间件相关，详见[这里](/guide/v2/advanced/middleware)
* __router__ 路由规则相关，详见[这里](/guide/v2/advanced/router)
* __serviceWorker__ Service Worker 相关，详见[这里](/guide/v2/advanced/service-worker)

### 其他文件

除了上述目录，还有其他一些散落的文件。但因为修改的可能性较小，因此放在这里统一简述一下：

* server.dev.js, server.prod.js

    这两个文件作用于用户输入命令 `lavas dev` 和 `lavas start` 时配合 Lavas 使用的，用户一般不需要修改。Lavas 集成的命令简介将在[ Lavas 命令](/guide/v2/basic/cli)中介绍。

* .babelrc, .editorconfig, .fecsignore, .fecsrc, .gitignore

    这些文件实际上并不属于 Lavas，而是项目通用文件，分别处理 babel 对 es6/7 的转码规则，IDE 的配置，fecs 代码风格检查的配置和 git 的忽略文件，开发者一般不需要修改。

* LINCENSE, package.json, README.md

    这些文件也不属于 Lavas，开发者可以根据自己情况自行修改。

### .lavas

细心的开发者可能注意到了，一旦 Lavas 项目运行过一次，根目录下还会增加一个 `/.lavas` 目录。这个目录是 Lavas 在运行之前事先生成的，如之前提到过的自动生成的路由规则，以及处理热加载 (hotreload) 等都放置在这里，开发者__不应该__修改这个目录的内容，并且将目录留在 `/.gitignore` 中，防止协作开发时互相影响。

## Lavas 自动路由生成方法

Lavas 2.0 的一大功能点是能够根据 `/pages/` 目录的 vue 文件和层级自动生成路由规则，帮助开发者快速搭建站点，避免繁琐且重复的路由规则配置。因此这里我们来了解一下 Lavas 是如何自动生成路由规则的。

### 普通情况

绝大部分情况下，Lavas 会直接根据 `/pages/` 中的 vue 文件层级生成对应的路由规则。此外因为 vue 建议的命名规范，一般 vue 文件都会命名为首字母大写的形式(如 `/pages/Appshell.vue` )，而路由规则一般均采用小写，所以这里 Lavas 会将 vue 文件名的首字母__改为小写__处理，但__不会__处理中间目录的大小写。

举例来说

```
/pages/Appshell.vue => /appshell
/pages/Error.vue => /error
```

### Index.vue 的特殊处理

存在有一类特殊情况：当 vue 文件命名为 Index.vue 时，开发者可能是想让它担任首页(至少是某个模块内的首页)的角色。因此这种情况下 Lavas 会进行特殊处理，去掉这里的 `index`。

举例来说

```
/pages/Index.vue => /
```

__注意：__ 如 `/pages/Index.vue` 时依然采用 `/index` 是__无法__访问到目标页面的！

### 动态参数

有些页面需要一个动态参数来进行访问。例如某博客站点需要展现某篇博客的内容详情，它的路由规则就很可能是 `/detail/[id]`， 其中 `[id]` 可能是数字，用来标识博客文章的 ID。这种情况我们需要用到 Lavas 动态参数功能。

我们可以在 `/pages/detail/` 目录中建立一个 vue 文件并命名为 `_id.vue` (下划线开头表示动态参数)。在这个 vue 文件中，我们可以通过 `this.$route.params.id`( script 中 )或者 `{{$route.params.id}}`( template 中 ) 获取这个参数。

### 嵌套路由

Lavas 同样支持 vue-router 的[嵌套路由](https://router.vuejs.org/zh-cn/essentials/nested-routes.html)。开发者需要在 `/pages` 目录中新建两个同名的目录和 Vue 文件(但首字母大小写不同，目录小写，文件大写)即可实现。

Vue 文件作为父路由的框架页面，包含 `<router-view>` 节点；目录内的其他 Vue 文件作为每个子路由的页面，填充在父路由的 `<router-view>` 节点内。

举例来说，我们有一套商品详情页面拥有相同的头部，通过嵌套路由来实现是，目录结构大约是：(尤其注意 detail 的大小写)

```
lavas-project
└── pages/
    ├── Detail.vue
    └── detail/
        ├── Product.vue
        ├── Service.vue
        └── Mixed.vue
```

`Detail.vue` 中需要包含 `<router-view>` 节点，如下：
```html
<template>
    <header>xxx</header>
    <router-view></router-view>
</template>
```

`detail/` 目录内部的 Vue 文件和普通的没有太大差别，正常开发即可。

在这种情况下，生成的路由会是 `/detail/product`, `/detail/service` 等等，用 vue-router 的 `router.js` 来描述的话，大概是：

```javascript
let routes = [{
    path: '/detail',
    name: 'detail',
    component: xxx,
    children: [{
        path: 'product',
        name: 'detailProduct',
        component: xxx
    },{
        path: 'service',
        name: 'detailService',
        component: xxx
    },{
        path: 'mixed',
        name: 'detailMixed',
        component: xxx
    }]
}]
```
