# Lavas 的 App Shell 模型

App Shell 模型是架构 PWA 的一种方式，它能够可靠且即时地让站点快速加载到用户屏幕上，获得与本地 APP 相似的体验。

![appshell](https://lavas.baidu.com/doc-assets/pwa-doc/architecture/images/appshell.png)

关于 App Shell 本身的概念和优势可以参考 Lavas 官网的[文章](https://lavas.baidu.com/doc/architecture/the-app-shell-model)，这里不作展开。

如果您阅读过[为 Vue 项目添加骨架屏](https://zhuanlan.zhihu.com/p/28465598)一文，可能已经了解了骨架屏 (Skeleton)的作用其实和 App Shell 非常类似，解决的都是在页面加载时尽量缩短白屏，给用户提供更好的页面切换体验。

在实际实现的过程中，我们发现根据渲染模式(SPA/MPA 和 SSR)的不同，两者各有问题又互相补充，因此 Lavas 在 SPA/MPA 模式下使用 Skeleton，在 SSR 模式下使用 App Shell。下面将分别讲述它们的配置和使用方式。

## SPA/MPA 模式下的 Skeleton

如果您关注过构建后的入口 HTML 页面，应该发现 Skeleton 的内容会被直接注入到页面中。换言之，在页面的 HTML 结构中就已经包含了 Skeleton，后续渲染成功后再进行替换。如果配合 Service Worker，则可以将这些入口的 HTML 文件缓存起来，在页面切换时快速展现，达成目标。

我们的思路是：把每个入口 HTML 添加到 Service Worker 的预缓存列表中，并且告知 Service Worker在请求某个页面时，从缓存中把对应的 HTML 返回出来供前端直接展现。后续的加载和替换 Skeleton 由 js 继续走正常流程完成，就无需我们额外关心了。

在 [Lavas 的入口](/guide/v2/advanced/entry)章节我们已经介绍过修改入口中的 Skeleton.vue 完成骨架本身的样式编写。因此这里我们主要聚焦在 Service Worker 的相关部分。

### SPA 下 Service Worker 的配置方式

在 SPA/MPA 的模式下，构建生成的最终目录中会显式地存在入口 HTML 文件，其中也包括了 Skeleton 的内容。我们假设我们的项目仅有一个入口 `main`，因此它是一个 SPA。在构建完成后的目录中应该存在一个 `main.html`。我们先把它加到 Service Worker 的预缓存列表中去，参照 [Lavas 中的 Service Worker](/guide/v2/advanced/service-worker) 章节，在 `/lavas.config.js` 中的 `serviceWorker` 部分作如下配置：

```javascript
serviceWorker: {
    // ...
    globPatterns: [
        '**/*.{html,js,css,eot,svg,ttf,woff}'
    ],
    // ...
}
```

将 `*.html` 包括其中，即可让 `main.html` 包含在预缓存列表中了。下一步我们需要告知 Service Worker 在切换页面时 (或称 navigation 请求)使用这个 HTML 作为响应。在 `/core/service-worker.js` 最后添加

```javascript
workboxSW.router.registerNavigationRoute('/main.html');
```

`registerNavigationRoute` 方法是 WorkBox 提供的一个快捷方法 ([API](https://developers.google.com/web/tools/workbox/reference-docs/latest/module-workbox-sw.Router#registerNavigationRoute))，它的作用是在 navigation 请求时使用参数所示的内容(`/main.html`)作为响应返回，而不是真正发起网络请求。因此在 SPA 的情况下，所有页面加载之前都会由 Service Worker 返回入口文件。而其中包含的 Skeleton 则替代了白屏，成为提升体验的关键。

### MPA 下 Service Worker 的配置方式

MPA 和 SPA 的差别在于使用了多个入口。因此构建生成的最终目录也会存在多个入口 HTML 文件。将它们添加到预缓存列表的方法和 SPA 相同，使用 `*.html` 即可。所以问题的核心在于 `registerNavigationRoute` 这一步如何分情况使用不同的 HTML。

我们假设在上述 main 入口之外，还存在一个 user 入口，其对应 URL 为 `/user/*`。我们可以利用 `registerNavigationRoute` 的 `blacklist` 和 `whitelist` 参数实现这个判断。

```javascript
workboxSW.router.registerNavigationRoute('/main.html');
workboxSW.router.registerNavigationRoute('/user.html', {
    whitelist: [/^\/user/]
});
```

之前提过，WorkBox 的注册路由是__越靠后越优先匹配__，因此我们将范围较小的 user 写在后面，并且使用 `whitelist` 限定它的作用范围；而剩余的路由就都落到了 main 中。

## SSR 模式下的 App Shell

Skeleton 之所以没法在 SSR 模式下生效，原因主要有这么几个：

1. 构建完成的最终目录中并不存在独立的 HTML 入口文件，所以无法将其添加到预缓存文件中
2. 服务端返回的 HTML 是每次请求最后由 Vue SSR 调用 `renderToString` 渲染出来的，这里面并不包括 Skeleton
3. 所有首屏请求都由服务端渲染，假设将这些返回都以 HTML 的形式存储到缓存中，那么随着 URL 的细微变化，缓存的数量是无穷的

我们虽然无法像 SPA/MPA 那样把所有有限的入口 HTML 都缓存起来，但我们可以把他们共同的外壳 (App Shell) 剥离出来并缓存起来，之后每次请求都返回这个外壳，再进行前端渲染，就可以实现和 SPA/MPA 相同的效果了。

![SSR App Shell](http://boscdn.bpc.baidu.com/assets/lavas/codelab/appshell.png)

在这种模式下，只要缓存中存在这个外壳，程序就没有请求服务端 HTML 的必要，只需要请求 API 接口获取数据即可。如果开发者的应用对于 API 的实时性要求不高，甚至 API 请求也可以进行缓存。无论 API 如何处理，对服务端来说，使用 Vue SSR 流程处理 navigation 请求仅限于浏览器缓存中不存在外壳时，也就是第一次访问时。后续访问即便是刷新页面，因为外壳的存在也不会再走 SSR 流程了，这和传统的 SSR 模式是不同的。

关于这个模式，Lavas 在知乎专栏的文章[SSR 架构项目实现离线可用（思路&案例）](https://zhuanlan.zhihu.com/p/30791448)有更详细的介绍。

补充一点，App Shell 和 Skeleton 的差异在于，Skeleton 是直接存在于入口 HTML 中的，因此在缓存中我们能看到(一个或者多个)入口HTML；而 App Shell 是一个独立的页面，在缓存中我们只能看到一个独立的 App Shell HTML，不存在入口 HTML。

实现方式方面，和 Skeleton 模式相同，我们需要完成两个步骤：__添加预缓存列表__和__注册动态缓存__。但因为这里要添加的预缓存是一个路由 (如 `/appshell/main`)而非一个独立的文件，因此和 Skeleton 略有不同。我们使用 `/lavas.config.js` 中 `serviceWorker` 段的 `appshellUrls` 配置项进行声明，如下：

```javascript
// ...
serviceWorker: {
    // swSrc, swDest, globDirectory, globPatterns, globIgnores, dontCacheBustUrlsMatching
    appshellUrls: ['/appshell/main']
}
// ...
```

由于 WorkBox 的内部机制，需要为不存在在硬盘上的资源及其依赖关系进行声明，从而产生版本 revision 供其判断更新。但 Lavas 使这些内部机制对开发者透明，您只需要将 App Shell 的路径声明在这里即可，其他的由 Lavas 和 WorkBox 进行交互。(如果您对 WorkBox 的内部实现很感兴趣，可以参考[templatedUrls 配置项](https://developers.google.com/web/tools/workbox/reference-docs/latest/module-workbox-build#.Configuration))

接下来的注册动态缓存步骤和 SPA/MPA 是类似的，使用到的仍然是 `registerNavigationRoute` 方法，如下：

```javascript
// core/service-worker.js
workboxSW.router.registerNavigationRoute('/appshell/main');
```

如果您的应用有多个入口，需要使用多个 App Shell, 需要做的也仅仅是在 `appshellUrls` 数组中把所有外壳的 URL 都声明一下，_没有顺序要求__；然后在 `core/service-worker.js` 中调用多次 `registerNavigationRoute` 方法，和 SPA/MPA 基本一致，这里就不再重复了。

关于 SSR 模式下的 App Shell 模型更多更详细的信息还可以参考发表于 Lavas 知乎专栏的[在 Vue SSR 中使用 Service Worker](https://zhuanlan.zhihu.com/p/31630322)一文。
