# Lavas 中的 Service Worker

Service Worker 可以说是 PWA 中最能发挥开发者想象力和最复杂的部分。有关 Service Worker 本身的介绍可以移步 Lavas 官网的[相关章节](https://lavas.baidu.com/doc/offline-and-cache-loading/service-worker/service-worker-introduction)。

大体来说，在实际项目中，Service Worker 主要完成三个工作：

* __静态文件预缓存__ 能够提前预知的用户需要缓存的内容，通常是静态文件，例如 js, css, 字体文件等等。

* __动态缓存__ 用户在运行过程中实际发送请求后再进行缓存的内容，通常是动态的接口，因为含有动态参数所以不可能全部预缓存。动态缓存通常还有各类策略，如 networkFirst, cacheFirst 等等

* __appshell__  缓存页面的外部框架，在切换页面时先从缓存取出框架显示，再逐步渲染核心内容，从而提升加载性能和体验。这部分将在 [App Shell 模型](/v2/advanced/appshell)中详细讨论。

Lavas 提供了配置 + 模板的便捷方式帮助开发者生成 Serice Worker。以初始项目的配置为例，打开 `/lavas.config.js` 能看到 `serviceWorker` 这一段，如下：

```javascript
module.exports = {
    // ...
    serviceWorker: {
        swSrc: path.join(__dirname, 'core/service-worker.js'),
        swDest: path.join(BUILD_PATH, 'service-worker.js'),
        globDirectory: path.basename(BUILD_PATH),
        globPatterns: [
            '**/*.{html,js,css,eot,svg,ttf,woff}'
        ],
        globIgnores: [
            'sw-register.js',
            '**/*.map'
        ],
        appshellUrls: ['/appshell/main'],
        dontCacheBustUrlsMatching: /\.\w{8}\./
    },
    // ...
};
```

这些基本都是提供给 Lavas 内置的 WorkboxWebpackPlugin 使用的配置项。[WorkBox](https://github.com/GoogleChrome/workbox) 是 Google 推出的 sw-toolbox 和 sw-precache 的升级版，封装了一些常用的 API (如预缓存，动态缓存及常用策略等)，帮助开发者更简单快速地开发 Service Worker。而 WorkboxWebpackPlugin 则是 Workbox 的 webpack 插件，通过配置项和模板两部分来生成 service-worker.js。

## Service Worker 配置项

我们来看一下例子中使用的配置项(这些配置项基本都是必选的)。其余的可以参考 Workbox 的[官网](https://developers.google.com/web/tools/workbox/)

* __swSrc__

    生成 service-worker.js 所需的模板文件所在位置，后续会详细提及

* __swDest__

    生成的 service-worker.js 的存放位置。例子中放在了整体构建目录 (`/dist`) 的下面，即 `/dist/service-worker.js`

* __globDirectory__

    指定需要预缓存的静态文件的目录。例子设定为整体构建目录 (`/dist`)

* __globPatterns__

    相对于 globDirectory 指定的目录，指出哪些文件__需要__被预缓存。这里可以使用通配符，可以参考[node-glob#glob-primer](https://github.com/isaacs/node-glob#glob-primer)

* __globIgnores__

    相对于 globDirectory 指定的目录，指出哪些文件__不需要__被预缓存。和 globPatterns 一样，也可以使用通配符。service-worker.js 本身会被自动排除

* __appshellUrls__

    [App Shell 模型](/v2/advanced/appshell)文中会详细提及，这里先跳过

* __dontCacheBustUrlsMatching__

    Workbox 会将符合上述 glob 开头的三个配置项条件的所有静态文件逐个生成一个版本号 (称为 revision) 存入缓存，后续在面对同名文件时比较缓存中的版本号决定是否更新。但 Lavas 生成的静态资源文件绝大部分是在文件名中带有 hash 的 (如 `/dist/static/css/manifest.5e1ead3.js`)，一旦文件内容更新 hash 也会更新，因此 Workbox 内置的版本号就不再需要了，所以可以省略这个生成和比较的过程从而提升构建速度。因为 Lavas 生成的 hash 是8位的，所以例子中的正则也正匹配8位字母数字。

### 配置预缓存文件

将上述配置项中 glob 开头的三个配置完成，即可指明哪些文件需要被预缓存。一般来说常用的包括 html, js, css以及一些字体文件(如果使用了 iconfont 等字体实现小图标的类库的话)。而例如 map 文件 (用于方便查看混淆后的代码) 和 sw-register.js (用于注册 sw，如果预缓存住则后续无法更新 sw ) 则应该被排除在外。

通过这些配置，WorkboxWebpackPlugin 能够根据这些静态文件的信息生成 `service-worker.js` 并包含符合条件的预缓存文件。如果要实现动态缓存和 appshell，还需要 Service Worker 模板来进一步实现。

## Service Worker 模板

Service Worker 的模板位于 `/config/service-worker.js`。观察初始状态下的代码，我们可以发现在定义动态路由和 appshell 之前还有一些内容，如下：

```javascript
importScripts('/static/js/workbox-sw.prod.v2.1.2.js');

const workboxSW = new WorkboxSW({
    cacheId: 'lavas-cache',
    ignoreUrlParametersMatching: [/^utm_/],
    skipWaiting: true,
    clientsClaim: true
});

// Define precache injection point.
workboxSW.precache([]);

// doing something else ...
```

第一行的 `importScripts` 是引用 Workbox 的 API，这里的版本号是一个定制，由依赖的 WorkBox 版本决定。Lavas 已经进行了版本号锁定，所以开发者不必过于关注这里，维持原状即可。

第二段创建 WorkboxSW 实例的代码中，涉及到一些配置项。

* __cacheId__

    指定应用的缓存 ID，这会最终影响到缓存的名称。实际运行效果中，WorkBox 还会将域名加在缓存 ID 中共同作为缓存名称，因此重名的几率还是比较小的。

* __ignoreUrlParametersMatching__

    指明什么样的请求参数应该被忽略。Service Worker 的静态文件缓存会根据请求 URL 进行匹配。只要请求 URL 不同则认为是不同的资源。但有些参数出于统计使用，并不影响文件本身的内容，这类参数就应当被忽略。

    如例子中的 `utm_` 开头，是定义在 `manifest.json` 中的 `"start_url": "/?utm_source=homescreen"`，它不应该影响 Service Worker 对于文件缓存的判断。

* __skipWaiting__

    在 Service Worker 的 install 阶段完成后无需等待，立即激活 (activate)。作用等同于 `self.skipWating()`

* __clientsClaim__

    在 Service Worker 的 activate 阶段让所有没被控制的页面受控。作用等同于 `self.clients.claim()`

    同时使用 `skipWaiting` 和 `clientsClaim` 可以让 Service Worker 在下载完成后立即生效

初始化 WorkboxSW 这个类的其他参数可以参考[WorkBox 文档](https://developers.google.com/web/tools/workbox/reference-docs/latest/module-workbox-sw.WorkboxSW)。

第三段的 `workboxSW.precache([]);` 看似是一句空语句，其实是一个代码插入点，刚才提过的预缓存的文件会经过 WorkboxWebpackPlugin 自动插入到这里，如果你感兴趣的话可以尝试看看构建后的 `/dist/service-worker.js`。

在这些准备工作之后，下面就是开发者发挥的空间了。

### 设置动态缓存规则

```javascript
// Define runtime cache.
workboxSW.router.registerRoute(new RegExp('https://query\.yahooapis\.com/v1/public/yql'),
    workboxSW.strategies.networkFirst());
```

Workbox 提供的 `resigerRoute` 方法接受两个参数，第一个是匹配请求 URL 的正则表达式，第二个是内置的缓存策略。除了例子中的 networkFirst，Workbox 还提供了 networkOnly, cacheFirst, cacheOnly, staleWhileRevalidate等等。关于这个方法的详细情况请参见 [API](https://developers.google.com/web/tools/workbox/reference-docs/latest/module-workbox-sw.Router#registerRoute)

经过这条配置，每次请求的 URL 如果匹配这个正则(其实是雅虎天气获取接口)， 在返回数据时会将数据进行缓存。如果网络连接故障，则返回缓存内容。配合预缓存了所有静态文件，站点就拥有了离线访问能力！

如果开发者对于每个缓存策略的含义还不清楚，可以参考 [The Offline Cookbook](https://jakearchibald.com/2014/offline-cookbook/#serving-suggestions-responding-to-requests), 也可以逐个参考 WorkBox 的策略 API，如 [CacheFirst](https://developers.google.com/web/tools/workbox/reference-docs/latest/module-workbox-runtime-caching.CacheFirst) 等。

### 跨域资源的小坑

当请求的是跨域资源(不仅限于接口，也包括图片等)并且目标服务器并没有设置 CORS 时，响应类型会被设置为 `'opaque'` 并且 HTTP 状态码会被设置为 `0`。出于安全考虑，WorkBox 对于这类资源的信任度不高，在使用 CacheFirst 策略时只缓存 HTTP 状态码为 `200` 的资源。所以这类资源不会被缓存，当然在离线时也无法被展现了。

如果开发者想使用跨域的资源且目标站点不支持 CORS，为了缓存下来，我们还需要额外配置合法的 HTTP 状态码，如下：

```javascript
workboxSW.router.registerRoute(/^https:\/\/ss\d\.baidu\.com/i,
    workboxSW.strategies.cacheFirst({
        cacheableResponse: {
            statuses: [0, 200]
        }
    })
);
```

这样状态码为 `0` 或者 `200` 的资源都会被缓存，达成了我们的需求。

### 动态缓存的注册顺序

当注册了多个动态缓存之后，如果被注册的正则存在交集，则还存在一个匹配顺序的问题。

WorkBox 的内部使用一个数组记录所有动态缓存的正则表达式。在开发者使用 `registerRoute` 时，内部调用数组的 `unshift` 方法进行扩充。因此，越往后的路由规则将存在于数组越靠前的位置。而在匹配时，是按数组从前到后的顺序进行匹配并响应的。因此结论是：__越后注册的规则将越先匹配__。

举例来说

```javascript
workboxSW.router.registerRoute(/^https:\/\/ss\d\.baidu\.com/i,
    workboxSW.strategies.cacheFirst({
        cacheableResponse: {
            statuses: [0, 200]
        }
    })
);

workboxSW.router.registerRoute(/^https:\/\/.*\.baidu\.com/i,
    workboxSW.strategies.networkOnly());
```

这种配置下，访问 `*.baidu.com` 的所有请求都会命中第二条规则，从而使用 networkOnly 规则，所以__不会__缓存任何文件。更换两者的注册顺序可以解决这个问题。

## 注册 Service Worker

Lavas 会自动注册生成的 Service Worker，所以一般情况下不需要开发者额外关注。如果您有兴趣了解注册方式，这一部分会大概介绍一下。

Lavas 内部使用 webpack 进行构建，其中处理 Service Worker 的注册问题时使用一个名为 [sw-register-webpack-plugin](https://github.com/lavas-project/sw-register-webpack-plugin) 的插件(也由 Lavas 开发组进行开发)。这款插件的作用有两个：

1. 在生成目录(默认 `/dist`) 生成 `sw-register.js`，用以注册 Service Worker

2. 是在编译时找到 HTML 文件，在 `</body>` 标签之前插入一段代码，用来引入 `sw-register.js`

我们从这两步分别了解一下这个插件。

### sw-register.js

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

### 引入 sw-register.js

上面提过，sw-register-webpack-plugin 会在 HTML 文件中寻找 `</body>` 标签并插入内容，因此这里需要明确，只有 SPA/MPA 模式才会生成 HTML 入口文件，也就是说：__插件只在 SPA/MPA 模式下插入内容__，SSR 因为没有独立的入口 HTML 文件生成，因此采用别的方案，这个将在后面讨论。

插件插入的内容大致如下：

```
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

因为 SSR 没有单独的入口 HTML 文件生成，因此 Lavas 需要完成插件的一部分工作，即引入 `sw-register.js`。SSR 需要给 renderer 提供一个 index.html 作为服务端模板，在这个 index.html 的底部，额外加上代码即可。

最后重申一点，所有和注册 Service Worker 相关的工作都已经由 Lavas 自动完成，这里仅仅是表述内部的做法，并不需要开发者额外进行任何配置或者开发。
