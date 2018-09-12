# Lavas 中的 Service Worker

> warn
>
> lavas-core-vue@1.2.0 版本开始使用 workbox@3.x，模板部分和配置项发生了一定的变化。本篇文档将以最新版本进行描述，如果您还在使用 lavas-core-vue@1.1.x (即 workbox@2.x )，可以查看[旧版文档](/guide/v2/advanced/service-worker-workbox-2)

Service Worker 可以说是 PWA 中最能发挥开发者想象力和最复杂的部分。有关 Service Worker 本身的介绍可以移步 Lavas 官网的[什么是 Service Worker](https://lavas.baidu.com/doc/offline-and-cache-loading/service-worker/service-worker-introduction)。

大体来说，在实际项目中，Service Worker 主要完成三个工作：

* __静态文件预缓存__ 能够提前预知的用户需要缓存的内容，通常是静态文件，例如 js, css, 字体文件等等。

* __动态缓存__ 用户在运行过程中实际发送请求后再进行缓存的内容，通常是动态的接口，因为含有动态参数所以不可能全部预缓存。动态缓存通常还有各类策略，如 networkFirst, cacheFirst 等等

* __appshell__  缓存页面的外部框架，在切换页面时先从缓存取出框架显示，再逐步渲染核心内容，从而提升加载性能和体验。这部分将在 [Skeleton 和 App Shell 模型](/guide/v2/advanced/appshell)中详细讨论。

初始化生成的项目默认已经带有 Service Worker。Lavas 的 Service Worker 可以分为两部分：

1. __配置部分__

    负责一些基本项的配置，如模板位置，生成位置等等。

2. __模板部分__

    主要处理__动态缓存__和 __appshell__。

## Service Worker 配置项

以初始项目的配置为例，打开 `/lavas.config.js` 能看到 `serviceWorker` 这一段，如下：

```javascript
module.exports = {
    // ...
    serviceWorker: {
        enable: true,
        swSrc: path.join(__dirname, 'core/service-worker.js'),
        swDest: path.join(BUILD_PATH, 'service-worker.js'),
        appshellUrl: '/appshell'
    },
    // ...
};
```

> info
>
>这些基本都是提供给 Lavas 内置的 WorkboxWebpackPlugin 使用的配置项。[WorkBox](https://github.com/GoogleChrome/workbox) 是 Google 推出的 sw-toolbox 和 sw-precache 的升级版，封装了一些常用的 API (如预缓存，动态缓存及常用策略等)，帮助开发者更简单快速地开发 Service Worker。而 WorkboxWebpackPlugin 则是 Workbox 的 webpack 插件，通过配置项和模板两部分来生成 service-worker.js。


我们来看一下例子中使用的配置项(这些配置项基本都是必选的)。其余的可以参考 Workbox 的[官网](https://developers.google.com/web/tools/workbox/)

* __enable__

    是否启用 Service Worker，默认为 `true`。

* __swSrc__

    生成 service-worker.js 所需的模板文件所在位置，后续会详细提及

* __swDest__

    生成的 service-worker.js 的存放位置。例子中放在了整体构建目录 (`/dist`) 的下面，即 `/dist/service-worker.js`

* __swPath__

    生成的 service-worker.js 在 sw-register.js 中默认会使用 publicPath 进行完整可访问路径拼接，如果您需要指定一个专有的 service-worker.js 文件的可访问 path，可以通过 `swPath` 配置指定，该配置字段默认不开启。

* __appshellUrl__

    [Skeleton 和 App Shell 模型](/guide/v2/advanced/appshell)文中会详细提及，这里先跳过

### 预缓存文件列表

workbox-webpack-plugin@3.x 会自动把 webpack 处理的 __所有静态文件__ 列为预缓存文件。在构建完成后会单独保存在一个 precacheList 文件中，以 JSON 的格式。

如果想对这些文件进行进一步的控制（例如增加额外的，或者删除无用的）需要使用一些高级的配置项，可以查阅 workbox 官网的[这篇文档](https://developers.google.com/web/tools/workbox/modules/workbox-webpack-plugin#configuration)

通过这些配置，WorkboxWebpackPlugin 能够根据这些静态文件的信息生成 `service-worker.js` 并包含符合条件的预缓存文件。如果要实现动态缓存和 appshell，还需要 Service Worker 模板来进一步实现。

## Service Worker 模板

Service Worker 的模板位于 `/core/service-worker.js`。观察初始状态下的代码，我们可以发现在定义动态路由和 appshell 之前还有一些内容，如下：

```javascript

workbox.core.setCacheNameDetails({
    prefix: 'lavas-cache',
    suffix: 'v1',
    precache: 'install-time',
    runtime: 'run-time',
    googleAnalytics: 'ga'
});

workbox.skipWaiting();
workbox.clientsClaim();

workbox.precaching.precacheAndRoute(self.__precacheManifest || []);

// doing something else ...
```

第一段设置一些缓存名称的配置项。相当于原先 workbox 2.x 的构造函数。您也可以从[官网文档](https://developers.google.com/web/tools/workbox/reference-docs/latest/workbox.core#.setCacheNameDetails)中获取更多信息

* __prefix__

    指定应用的缓存前缀，同时应用于预缓存和动态缓存的名称，拼接在最前面。

* __suffix__

    指定应用的缓存后缀，同时应用于预缓存和动态缓存的名称，拼接在最后面。

* __precache__

    指明预缓存使用的缓存名称

* __runtime__

    指定动态缓存使用的缓存名称

* __googleAnalytics__

    `workbox-google-analytics` 使用的缓存名称。关于 workbox 和 google analytics 之间的配合，可以查阅[这里](https://developers.google.com/web/tools/workbox/modules/workbox-google-analytics)

第二段的两句 (`workbox.skipWaiting();` 和 `workbox.clientsClaim();`) 一般共同使用，使得 Service Worker 可以在 activate 阶段让所有没被控制的页面受控，让 Service Worker 在下载完成后立即生效

第三段的 `workbox.precaching.precacheAndRoute(self.__precacheManifest || []);` 使用到的 `self.__precacheManifest` 是定义在单独的一个预缓存文件列表中。如前所述，这个列表包含 webpack 构建过程中的所有静态文件。而这里就是告诉 workbox 把这些文件预缓存起来。

在这些准备工作之后，下面就是开发者发挥的空间了。

### 设置动态缓存规则

```javascript
// Define runtime cache.
workbox.routing.registerRoute(/^https:\/\/query\.yahooapis\.com\/v1\/public\/yql/,
    workbox.strategies.networkFirst());
```

> info
>
> Workbox 提供的 `resigerRoute` 方法接受两个参数，第一个是匹配请求 URL 的正则表达式，第二个是内置的缓存策略。除了例子中的 networkFirst，Workbox 还提供了 networkOnly, cacheFirst, cacheOnly, staleWhileRevalidate等等。关于这个方法的详细情况请参见 [API](https://developers.google.com/web/tools/workbox/reference-docs/latest/workbox.routing#.registerRoute)

> 经过这条配置，每次请求的 URL 如果匹配这个正则(其实是雅虎天气获取接口)， 在返回数据时会将数据进行缓存。如果网络连接故障，则返回缓存内容。配合预缓存了所有静态文件，站点就拥有了离线访问能力！

> 如果开发者对于每个缓存策略的含义还不清楚，可以参考 [The Offline Cookbook](https://jakearchibald.com/2014/offline-cookbook/#serving-suggestions-responding-to-requests) 或者 workbox 上也有[简述](https://developers.google.com/web/tools/workbox/modules/workbox-strategies)。

### 缓存策略的参数

上述例子中，我们直接使用了 `networkFirst()`，没有参数。但实际上，每位开发者都可能会有一些个性化的配置，对策略进行更精细化的控制，例如：

1. 使用一个特定的缓存 (指定一个不一样的缓存名称)
2. 设置缓存失效时间或者个数上限

这些需求都可以通过缓存的参数来实现。主要有两种：

1. `cacheName`： 指定新的缓存名称，使得符合这条正则的请求的缓存全都存到一起
2. `plugins`：指定插件的数组。插件可以实现缓存失效时间或者个数上限，也包括其他的功能，甚至可以自定义。

### 跨域资源的小坑

当请求的是跨域资源(不仅限于接口，也包括图片等)并且目标服务器并没有设置 CORS 时，响应类型会被设置为 `'opaque'` 并且 HTTP 状态码会被设置为 `0`。出于安全考虑，WorkBox 对于这类资源的信任度不高，在使用 CacheFirst 策略时只缓存 HTTP 状态码为 `200` 的资源。所以这类资源不会被缓存，当然在离线时也无法被展现了。

如果开发者想使用跨域的资源且目标站点不支持 CORS，为了缓存下来，我们还需要额外配置合法的 HTTP 状态码。这里就需要用到上面提到的 `plugins`了，如下：

```javascript
workbox.routing.registerRoute(/^https:\/\/ss\d\.baidu\.com/i,
    workbox.strategies.cacheFirst({
        plugins: [
            new workbox.cacheableResponse.Plugin({
              statuses: [0, 200]
            })
        ]
    })
);
```

这样状态码为 `0` 或者 `200` 的资源都会被缓存，达成了我们的需求。

### 动态缓存的注册顺序

当注册了多个动态缓存之后，如果被注册的正则存在交集，则还存在一个匹配顺序的问题。

WorkBox 的内部使用一个数组记录所有动态缓存的正则表达式。在开发者使用 `registerRoute` 时，内部调用数组的 `unshift` 方法进行扩充。因此，越往后的路由规则将存在于数组越靠前的位置。而在匹配时，是按数组从前到后的顺序进行匹配并响应的。因此结论是：__越后注册的规则将越先匹配__。

举例来说

```javascript
workbox.routing..registerRoute(/^https:\/\/ss\d\.baidu\.com/i,
    workbox.strategies.cacheFirst({
        cacheableResponse: {
            statuses: [0, 200]
        }
    })
);

workbox.routing..registerRoute(/^https:\/\/.*\.baidu\.com/i,
    workbox.strategies.networkOnly());
```

这种配置下，访问 `*.baidu.com` 的所有请求都会命中第二条规则，从而使用 networkOnly 规则，所以__不会__缓存任何文件。更换两者的注册顺序可以解决这个问题。

## 注册 Service Worker (扩展)

*提示：这部分内容由 Lavas 内部处理，并不需要开发者进行参与，仅仅作为解答开发者疑问的扩展阅读存在。*

Service Worker 编写完成后，还需要进行注册才能真正生效。常规的注册代码能够在各类 Service Worker 教程或文章中找到，但在实际项目中有一个不得不考虑的问题，使得我们必须对注册代码进行一些改动，那就是 __Service Worker 更新__ 的问题。

### 解决思路

为了最大化利用浏览器缓存 `service-worker.js`，但又保证一旦项目更新时浏览器能够及时更新之，Lavas 的解决思路是：

1. 将注册代码单独放置在 `sw-register.js` 中

2. `sw-register.js` 中实际注册 `service-worker.js` 的部分，在后面添加 `?v=xxxx`，取值为编译时间。因此一次编译后不会修改，`service-worker.js` __可以__ 被浏览器缓存。

3. 在 HTML 中引用 `sw-register.js`，同样在后面添加 `?v=xxxx`，但这里取值为当前时间，因此每次请求都在变化，__避免__ 浏览器对 `sw-register.js` 进行缓存。

这样每次浏览器都会重新请求 `sw-register.js`。如果重新编译，`sw-register.js` 中注册的 `service-worker.js?v=xxxx` 的 `v` 会变化，迫使浏览器重新请求；如果未重新编译，那么这个 `v` 不会变化，浏览器可以直接使用缓存中的 `service-worker.js`。

### 实现方式

Lavas 内部使用 webpack 进行构建，其中处理 Service Worker 的注册问题时使用一个名为 [sw-register-webpack-plugin](https://github.com/lavas-project/sw-register-webpack-plugin) 的插件(也由 Lavas 开发组进行开发)。这款插件的作用有两个：

1. 在生成目录(默认 `/dist`) 生成 `sw-register.js`，用以注册 Service Worker

2. 是在编译时找到 HTML 文件，在 `</body>` 标签之前插入一段代码，用来引入 `sw-register.js`

我们从这两步分别了解一下这个插件。

#### sw-register.js

生成的 `sw-register.js` 大致内容如下，其中的参数 `v` 以编译的时间生成时间戳，保证获取的 `service-worker.js` 不受浏览器缓存的影响。

```javascript
if ('serviceWorker' in navigator) {
    // 例如v=20171205175126
    navigator.serviceWorker.register('/service-worker.js?v=xxxx').then(function(reg) {
        reg.onupdatefound = function() {
            var installingWorker = reg.installing;
            installingWorker.onstatechange = function() {
                switch (installingWorker.state) {
                    case 'installed':
                        if (navigator.serviceWorker.controller) {
                            var event = document.createEvent('Event');
                            event.initEvent('sw.update', true, true);
                            window.dispatchEvent(event);
                        }
                        break;
                }
            };
        };
    }).catch(function(e) {
        console.error('Error during service worker registration:', e);
    });
}
```

从这个文件内容来看，它的主要工作包括：

1. 调用 `navigator.serviceWorker.register` 注册 Service Worker

2. 注册 `updatefound` 事件并监听 Service Worker 的更新，并在更新时分发 `'sw.update'` 事件

补充说明：这个 `'sw.update'` 事件在 Lavas 项目下 `/components/UpdateToast.vue` 组件进行监听，并在更新时弹出提示，引导用户刷新页面。

#### 引入 sw-register.js

上面提过，sw-register-webpack-plugin 会在 HTML 文件中寻找 `</body>` 标签并插入内容，因此这里需要明确，只有 SPA 模式才会生成 HTML 文件，也就是说：__插件只在 SPA 模式下插入内容__，SSR 因为没有独立的 HTML 文件生成，因此采用别的方案，这个将在后面讨论。

插件插入的内容大致如下：

```html
<script>
window.onload = function () {
    var script = document.createElement('script');
    var firstScript = document.getElementsByTagName('script')[0];
    script.type = 'text/javascript';
    script.async = true;
    script.src = '/sw-register.js?v=' + Date.now();
    firstScript.parentNode.insertBefore(script, firstScript);
};
</script>
```

作用也很明显，在整个 HTML 的第一个 `<script>` 之前插入新的 `<script>` 引用 `sw-register.js`，同样通过时间戳来屏蔽浏览器的缓存。但这里的 `v` 和 `sw-register.js` 里的 `v` 有区别，`sw-register.js` 中的 `v` 在编译一次之后就确定并写入文件(如果有兴趣你可以查看 `/dist/sw-register.js`)，之后不会再改变；而这里的值是 `Date.now()`，所以每次都请求新的 `sw-register.js`。依靠这种模式，可以以较小的代价在第一时间更新到最新的 Service Worker。

### SSR 模式下引入 sw-register.js

因为 SSR 没有单独的 HTML 文件生成，因此 Lavas 需要完成插件的一部分工作，即引入 `sw-register.js`。SSR 需要给 renderer 提供一个 index.html 作为服务端模板，在这个 index.html 的底部，额外加上代码即可。

最后重申一点，所有和注册 Service Worker 相关的工作都已经由 Lavas 自动完成，这里仅仅是表述内部的做法，并不需要开发者额外进行任何配置或者开发。
