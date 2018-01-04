# Skeleton 和 App Shell 模型

App Shell 模型是架构 PWA 的一种方式，它能够可靠且即时地让站点快速加载到用户屏幕上，获得与本地 APP 相似的体验。

![appshell](https://boscdn.baidu.com/assets/lavas/codelab/appshell-general.png)

关于 App Shell 本身的概念和优势可以参考 Lavas 官网的[文章](https://lavas.baidu.com/doc/architecture/the-app-shell-model)，这里不作展开。

如果您阅读过[为 Vue 项目添加骨架屏](https://zhuanlan.zhihu.com/p/28465598)一文，可能已经了解了骨架屏 (Skeleton)的作用其实和 App Shell 非常类似，解决的都是在页面加载时尽量缩短白屏，给用户提供更好的页面切换体验。

在实际实现的过程中，Lavas根据渲染模式 (SPA 和 SSR) 的不同，分别使用两种不同的解决方案。__在 SPA 模式下使用 Skeleton，在 SSR 模式下使用 App Shell__。下面将分别讲述它们的配置和使用方式。

## SPA 模式下的 Skeleton

如果您关注过构建后的 `index.html` 页面，应该发现 Skeleton 的内容会被直接注入到页面中。换言之，在页面的 HTML 结构中就已经包含了 Skeleton，后续渲染成功后再进行替换。如果配合 Service Worker，则可以将 HTML 文件缓存起来，在页面切换时快速展现，达成目标。

因此我们的思路是：__把 index.html 添加到 Service Worker 的预缓存列表中，并且告知 Service Worker在请求某个页面时，从缓存中把 HTML 返回出来供前端直接展现__。后续的加载和替换 Skeleton 由 js 继续走正常流程完成，就无需我们额外关心了。

为了实现思路，我们需要进行的操作有两个步骤：

1. Skeleton 本身的编写
2. 配置 Service Worker，添加预缓存文件列表

### 编写 Skeleton

Skeleton 的位置在 `/core/Skeleton.vue`。

```html
<template>
    <div class="skeleton-wrapper">
        <!-- skeleton content -->
    </div>
</template>

<script>
export default {
    name: 'skeleton'
};
</script>

<style lang="stylus" scoped>
// skeleton content style rules
</style>
```

`Skeleton.vue` 按照普通 Vue 页面组件的开发方式进行开发即可。一般情况在 `<template>` 中我们会使用一些与实际页面布局和颜色接近的图片来充当 Skeleton 的内容，替代加载时的白屏。如果图片尺寸不太大的话，这里尤其推荐使用 base64 编码直接写入，避免再进行网络请求，保证离线可用。

因为是个完整的 Vue 组件，所以还需要包括 `<script>`, `<style>` 两部分。`<script>` 方面，一般 Skeleton 是静态的，所以并不需要什么额外的操作；而 `<style>` 则把 Skeleton 的内容样式编写完整即可。

### 添加预缓存文件列表

下一步我们把构建生成的 `index.html` 加到 Service Worker 的预缓存列表中去，参照 [Lavas 中的 Service Worker](/guide/v2/advanced/service-worker) 章节，在 `/lavas.config.js` 中的 `serviceWorker` 部分作如下配置：

```javascript
serviceWorker: {
    // ...
    globPatterns: [
        '**/*.{html,js,css,eot,svg,ttf,woff}'
    ],
    // ...
}
```

将 `*.html` 包括其中，即可让 `index.html` 包含在预缓存列表中了。

## SSR 模式的 App Shell

Skeleton 之所以没法在 SSR 模式下生效，原因主要有这么两个：

1. 构建完成的最终目录中并不存在 `index.html`，所以无法将 Skeleton 加入其中，也无法将其添加到预缓存文件列表中
2. 所有首屏请求都由服务端渲染，假设将这些返回都以 HTML 的形式存储到缓存中，那么随着 URL 的细微变化，缓存的数量是无穷的

我们虽然无法像 SPA 那样把 `index.html` 缓存起来，但我们可以把他们共同的外壳 (App Shell) 剥离成独立路由，由 Service Worker 请求并缓存。之后拦截每次 HTML 请求都返回这个外壳，再进行前端渲染，就可以实现和 Skeleton 相同的效果了。

![SSR App Shell](https://boscdn.baidu.com/assets/lavas/codelab/appshell.png)

在这种模式下，只要缓存中存在这个外壳，程序就没有请求服务端 HTML 的必要，只需要请求 API 接口获取数据即可。(如果开发者的应用对于 API 的实时性要求不高，甚至 API 请求也可以进行缓存。) 无论 API 如何处理，对服务端来说，使用 Vue SSR 流程处理 HTML 请求仅限于浏览器缓存中不存在外壳时，也就是第一次访问时。后续访问即便是刷新页面，因为外壳的存在也不会再走 SSR 流程了，这和传统的 SSR 模式是不同的。

关于这个模式，Lavas 在知乎专栏的文章 [SSR 架构项目实现离线可用（思路&案例）](https://zhuanlan.zhihu.com/p/30791448) 和 [在 Vue SSR 中使用 Service Worker](https://zhuanlan.zhihu.com/p/31630322) 有更详细的介绍。

实现方式方面，和 Skeleton 模式类似，我们同样需要完成两个步骤：

1. App Shell 本身的编写
2. 配置 Service Worker，添加预缓存文件列表

### 编写 App Shell

首先我们需要开发一个 App Shell。我们创建一个 `/pages/Appshell.vue`，并编写如下内容：

```html
<template>
</template>

<script>
export default {
    name: 'appshell',
    metaInfo: {
        title: 'Lavas',
        meta: [
            {name: 'keywords', content: 'lavas PWA'},
            {name: 'description', content: '基于 Vue 的 PWA 解决方案，帮助开发者快速搭建 PWA 应用，解决接入 PWA 的各种问题'}
        ],
        bodyAttrs: {
            'empty-appshell': undefined
        }
    }
};
</script>
```

`metaInfo` 是 Lavas 内部使用的 [vue-meta](https://github.com/declandewet/vue-meta) 的默认配置 key，用来传递各类页面元信息，开发者可以根据项目的具体情况自行修改。其中最重要的 `empty-appshell` 虽然值是 `undefined`，但因为 Lavas 的内部判断机制，__不能修改或者删除__，否则将导致 App Shell 无法工作。

和 Skeleton 不同，App Shell 不需要编写内容和样式，而是直接复用全局框架的样式 (`/core/App.vue`)，因此这里不需要 `<style>`，`<template>` 内容也直接留空就可以。

### 配置 Service Worker

和 Skeleton 不同，这里要添加的预缓存并不是一个独立的文件，而是一个路由 (如 `/appshell`)。我们使用 `/lavas.config.js` 中 `serviceWorker` 段的 `appshellUrls` 配置项先进行声明，之后才可以正常使用。

```javascript
// ...
serviceWorker: {
    // swSrc, swDest, globDirectory, globPatterns, globIgnores, dontCacheBustUrlsMatching..
    appshellUrl: '/appshell'
}
// ...
```

### 多个 App Shell 的支持 (扩展)

默认情况 Lavas 会帮助开发者完成一个 App Shell 的注册。如果开发者有需求要支持多个 App Shell，需要额外进行一些操作。

相比单个 App Shell 的开发，多个 App Shell 的开发步骤有三个：

1. 多个 App Shell 本身的编写

    和单个 App Shell 基本相同，不再复述。

2. 配置 Service Worker，添加预缓存文件列表

    `appShellUrl` 参数 __只需要__ 填写适用面最广的 App Shell 路径。

    举例来说，存在两个 App Shell，A 适用于 `/user` 开头的路由，B 适用于剩余的路由。那么 B 就是适用面广的 App Shell，它的访问地址 (如 `/appshell/B` ) 应该被填入 `appShellUrl`

3. 编写 Service Worker 模板

Service Worker 模板位于 `/core/service-worker.js`，注册 App Shell 我们需要使用 WorkBox 的 `registerNavigationRoute` 方法，如下：

```javascript
workboxSW.router.registerNavigationRoute('/appshell/B', {
    whitelist: /^\/user/
});
```

`registerNavigationRoute` 方法是 WorkBox 提供的一个快捷方法 ([API](https://developers.google.com/web/tools/workbox/reference-docs/latest/module-workbox-sw.Router#registerNavigationRoute))，它的作用是在 HTML 请求 (`request.mode === 'navigate'`) 时使用参数内容 (`/appshell/B`) 作为响应返回，而不真正发起网络请求。因此现在所有页面加载之前都会由 Service Worker 返回 `index.html`，而其中包含的 Skeleton 则替代了白屏，成为提升体验的关键。

那为什么单个 App Shell 时不需要编写这句代码呢？

很简单，因为 Lavas 帮助开发者自动生成了这句代码。Lavas 获取 `appShellUrl` 参数并自动生成调用，如下：

```javascript
workboxSW.router.registerNavigationRoute('/appshell/A')
```

综上，开发者只需要把普适的 `/appshell/A` 编写在 `appShellUrl` 配置项中，再自行编写 `/appshell/B` 的注册即可。

## Skeleton 和 App Shell 的差异 (扩展)

*提示：这部分内容由 Lavas 内部处理，并不需要开发者进行参与，仅仅作为解答开发者疑问的扩展阅读存在。*

除了渲染模式不同之外，开发者是否存在一个疑问：为什么 Skeleton 需要把内容和样式都编写在内，而 App Shell 直接留空即可？

这和 Lavas 内部的构建机制以及 Vue 渲染页面有关。

* Skeleton

    Skeleton 虽然在源代码中以 Vue 页面组件存在，但在实际构建过程中通过内置的 [vue-skeleton-webpack-plugin](https://github.com/lavas-project/vue-skeleton-webpack-plugin) 将内容抽取出来，填入构建生成的 `index.html` 中。因此在构建完成的项目中并不能看到 Skeleton 单独存在，而是作为 HTML 的一部分。

    ```html
    <!-- dist/index.html -->
    <div id="app">
        <div class="skeleton-wrapper" data-v-xxxxx>
            <!-- skeleton content -->
        </div>
    </div>
    <script src="xxx">
    <script src="xxx">
    ```

    Vue 在实际渲染的时候，在渲染之前先展现 HTML 本来的内容，于是 Skeleton 被一并展现了；当渲染完成后，通过 `app.$mount('#app')` 将结果替换掉 `#app` 内本来的内容 (Skeleton)，表现为真实内容替换了 Skeleton。所以从这个角度不难理解，Skeleton 一定需要包含头尾等元素。

* App Shell

    和 Skeleton 相同，App Shell 也以 Vue 页面组件存在于源码。但因为处于 `/pages` 目录下，因此不但拥有单独路由，编译过后也能以独立文件存在。

    在 Vue 渲染时，它作为 `App.vue` 中 `<router-view>` 内部的内容被替换，而头尾是在 `App.vue` 中，`<router-view>` 之外的，因此 App Shell 不需要包含头尾。而因为页面未加载，数据请求也未发送，所以 App Shell 通常就是空的。
