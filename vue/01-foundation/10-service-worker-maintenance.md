# 维护 service worker 文件

开始之前，您可以查看 [servier worker](https://lavas.baidu.com/doc/offline-and-cache-loading/service-worker/01-service-worker-introduction) 相关内容，快速掌握相关基础。查看 service worker [ 浏览器支持情况](http://caniuse.com/#feat=serviceworkers)

servier-worker.js 作为缓存管理的重要文件，在导出工程的时候我们默认给了一个能覆盖缓存需求的 service-worker.js 文件。
但是我们默认提供的文件可能在后续您的开发过程中并不能完全覆盖您的需求，所以你需要对其进行一定的维护。

## service-worker.js

导出项目中，使用了 [service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) + [sw-precache](https://github.com/GoogleChrome/sw-precache) + [sw-precache-webpack-plugin](https://www.npmjs.com/package/sw-precache-webpack-plugin)( Webpack 插件)的方式，仅在 build 后自动生成可见的 service-worker.js 文件，可
* 支持离线缓存静态资源能力，通过配置实现动态网络缓存，以及文件更新机制
* 支持 service-worker.js 文件更新时，页面自动重载。

如果开发者没有特殊的缓存需求，可直接使用。如果开发者需要后续工程的定制化，就要再深入了解以下三方面内容：


## 如何配置缓存内容 —— 哪里配置，怎么配置 ？

开发者可通过 config/sw-precache.js 文件进行缓存配置，根据配置为用户缓存网站静态与动态资源，并截获用户的所有网络请求，决定是从缓存还是网络获取相应资源，限制缓存大小等。对于无额外需求的开发者，一般仅需配置该文件就可满足项目需求。

下面来看一下，具体配置结构（此处给出了一些常用配置，更全面的配置可通过 [sw-precache](https://github.com/GoogleChrome/sw-precache) 查看），该配置在 webpack.prod.conf.js 中被 [sw-precache-webpack-plugin](https://www.npmjs.com/package/sw-precache-webpack-plugin) 作为参数引入，build 时起作用，生成定制化 service-worker.js 文件。



``` js

/* sw-precache.js中的配置 */

build: {

    cacheId: 'my-vue-app',

    /* 生成的文件名称 */
    filename: 'service-worker.js',

    /* 需缓存的文件配置, 可以逐项添加
       需动态缓存的放到runtimeCaching中处理 */
    staticFileGlobs: [
        // 'dist/index.html',
        // 'dist/static/**/**.*'
    ],

    /* webpack生成的静态资源全部缓存 */
    mergeStaticsConfig: true,

    /* 忽略的文件 */
    staticFileGlobsIgnorePatterns: [
        /\.map$/ // map文件不需要缓存
    ],

    /* 需要省略掉的前缀名 */
    stripPrefix: 'dist/',

    /* 当请求路径不在缓存里的返回，对于单页应用来说，入口点是一样的 */
    navigateFallback: '/index.html',

    /* 白名单包含所有的.html (for HTML imports) 和
       路径中含’/data/’(for dynamically-loaded data). */
    navigateFallbackWhitelist: [/^(?!.*\.html$|\/data\/).*/],

    minify: true, // 是否压缩，默认不压缩

    maximumFileSizeToCacheInBytes: 4194304, // 最大缓存大小

    /* 生成service-worker.js的文件配置模板，不配置时采用默认的配置
        本demo做了sw的更新策略，所以在原有模板基础做了相应的修改 */
    templateFilePath: 'config/sw.tmpl.js',

    verbose: true,

    /* 需要根据路由动态处理的文件 */
    runtimeCaching: [
        {
            urlPattern: /\/material-design-icon/,
            handler: 'networkFirst'
        },

        /* 如果在staticFileGlobs中设置相同的缓存路径，可能导致此处不起作用 */
        {
            urlPattern: /\/fonts\//,
            handler: 'networkFirst',
            options: {
                cache: {
                    maxEntries: 10,
                    name: 'fonts-cache'
                }
            }
        }
    ]
}


// webpack.prod.conf.js中通过组件引入配置，生成文件
new SWPrecacheWebpackPlugin(config.swPrecache.build)


```



## 如何修改 service-worker.js 文件内容

**如果自动生成的文件实在无法满足项目需求，怎么进行定制化开发呢 ？**

要想找到答案，我们就要先去看看 sw-precache 工具是怎么生成了这个 `service-worker.js` 文件。


要让 sw-precahce 工具生成 service-worker.js 文件，需要给它提供一个模板文件。工具默认使用插件默认模板，但是您也可以定制自己的模板（最好参考默认模板），通过配置导入模板，实现定制化开发。在上面的配置文件可以发现，就是通过 `templateFilePath: 'config/sw.tmpl.js'` 导入定制化模板来生成`service-worker.js` 文件。
**项目中将模板文件提取到了 config 文件夹下，便于开发者后期相应的维护开发。**


**导出项目中做了什么定制化呢 ？**
这就来给大家介绍下，为了在 service-worker.js 文件内容更新时，能够让主页面及时做出重载更新，我们在 config/sw.tmpl.js 文件的 activate 监听事件中通过 postMessage 抛出了 'updateMessage' 的信息，在 sw-register.js 中，注册了消息的监听，一旦接收到 'updateMessage' 消息，主页面给出相应的更新提示。

**注意：** 在首次注册 service worker 时不发送更新信息，避免用户在首次进入页面时，就会再次重载，影响用户体验。


``` js
// sw.tmpl文件中
self.addEventListener('activate', function(event) {
    var setOfExpectedUrls = new Set(urlsToCacheKeys.values());

    event.waitUntil(
        caches.open(cacheName).then(function(cache) {
            return cache.keys().then(function(existingRequests) {
                return Promise.all(
                    existingRequests.map(function(existingRequest) {
                        if (!setOfExpectedUrls.has(existingRequest.url)) {
                            return cache.delete(existingRequest);
                        }
                    })
                );
            });
        }).then(function() {
            <% if (clientsClaim) { %>
            return self.clients.claim();
            <% } %>
        }).then(function() {
            if (!firstRegister) {
                return self.clients.matchAll()
                    .then(function (clients) {
                        if (clients && clients.length) {
                            var currentClient = clients[0];
                            currentClient.postMessage('updateMessage');
                        }
                    })
            }
        })
    );
});

```


## service worker 的注册


注册部分在项目的 src/sw-register.js 文件中，并在 `index.html` 引入执行。上面内容提及的 `service-worker.js` 更新时 `updateMessage` 的信息监听和页面重载部分，也是在 `src/sw-register.js`里完成的，开发者可根据需求做相应的扩展。


``` js
// sw-register.js文件, handlerUpdateMessage 对更新做出重新加载的处理
(function (window) {

    if ('serviceWorker' in navigator) {

        navigator.serviceWorker
            .register('/service-worker.js')
            .then(function (registration) {
                // console.log('Service Worker Registered');
            });

        navigator.serviceWorker.addEventListener('message', function (e) {
            // received the update message from sw
            if (e.data === 'updateMessage') {
                handlerUpdateMessage(e);
            }
        });

        var handlerUpdateMessage = function (e) {
            // 在这里可以检测到 service worker 文件的更新，通常我们建议做页面的 reload，开发者也可自定义一些处理，可参照官方模版给用户更新提示

            location.reload();
        };
    }
})(window);
```

``` js

// index.html 中引入注册代码
window.onload = function () {
    var script = document.createElement('script');
    var firstScript = document.getElementsByTagName('script')[0];
    script.type = 'text/javascript';
    script.async = true;
    script.src = '/sw-register.js?v=' + Date.now();
    firstScript.parentNode.insertBefore(script, firstScript);
};

```


## 缓存补充

缓存内容及策略主要通过 config/sw-precache.js 配置文件来控制，常用配置的参数如下：
* 配置项中有 `mergeStaticsConfig` 参数（定制化提供参数），默认是 true，即在没配置的情况下，默认缓存所有静态文件
* 如果不想缓存所有的静态文件，需要配置 `staticFileGlobs` 参数，将需要缓存的静态文件，依次写入
* 对于需要动态缓存的资源，可以通过配置文件中的 `runtimeCaching`  参数来配置，此时 sw-precache 模块就会帮我们引入 sw-toolbox 模块。所以在 sw-precache 中使用 runtimeCaching 配置选项可以参考 sw-toolbox 的配置规则，最终动态的路由规则会被添加到 service worker.js 文件的最后。
下面对 runtimeCaching 的具体配置也给出相应的介绍，后期开发应用还是比较广泛的。

例如，下面的配置为两种不同URL模式定义了实时缓存行为。它对两种请求使用不同的处理程序，并为/fonts/模式相匹配的请求指定了最大可用缓存：

```js
/* 需要根据路由动态处理的文件 */
runtimeCaching: [
    {
        urlPattern: /\/material-design-icon/,
        handler: 'fastest'
    },

    {
        urlPattern: /\/fonts\//,
        handler: 'networkFirst',
        options: {
            cache: {
                maxEntries: 10,
                name: 'fonts-cache'
            }
        }
    }
]
```

runtimeCaching 的配置选项数组中的每个对象都需要一个 urlPattern，它是一个正则表达式或一个字符串，遵循 sw-toolbox 的配置约定。此外，还需要一个 handler，指定其读取动态资源的策略（包括 5 种：networkFirst、cacheFirst、fastest、cacheOnly、networkOnly）。


下面配置相关参数介绍取自[使用指南](https://metaquant.org/programing/sw-precache-guide.html)， 为了您查阅方便直接备注在这里。

**sw-toolbox提供五种针对网络请求的处理程序(handler)，具体如下：**

* `networkFirst`：首先尝试通过网络来处理请求，如果成功就将响应存储在缓存中，否则返回缓存中的资源来回应请求。它适用于以下类型的API请求，即你总是希望返回的数据是最新的，但是如果无法获取最新数据，则返回一个可用的旧数据。

* `cacheFirst`：如果缓存中存在与网络请求相匹配的资源，则返回相应资源，否则尝试从网络获取资源。 同时，如果网络请求成功则更新缓存。此选项适用于那些不常发生变化的资源，或者有其它更新机制的资源。

* `fastest`：从缓存和网络并行请求资源，并以首先返回的数据作为响应，通常这意味着缓存版本则优先响应。一方面，这个策略总会产生网络请求，即使资源已经被缓存了。另一方面，当网络请求完成时，现有缓存将被更新，从而使得下次读取的缓存将是最新的。

* `cacheOnly`：从缓存中解析请求，如果没有对应缓存则请求失败。此选项适用于需要保证不会发出网络请求的情况，例如在移动设备上节省电量。

* `networkOnly`：尝试从网络获取网址来处理请求。如果获取资源失败，则请求失败，这基本上与不使用service worker的效果相同。

**sw-toolbox选项中的cache选项可以指定缓存的最大数目以及缓存时间等，具体如下：**

* `cache.name[String]`：用于存储实时缓存对象的缓存名称。使用唯一的名称允许您自定义缓存的最大空间和缓存时间。默认值：在运行时根据service worker的registration.scope值生成。

* `cache.maxEntries[Number]`：对缓存的项目实施least-recently缓存过期策略，可以将此项用于动态资源缓存。例如，将cache.maxEntries设置为10意味着在第11个项目被缓存之后，最近最少使用的条目将被自动删除。缓存永远不会超过cache.maxEntries规定的最大数量。此选项将仅在同时设置了cache.name时生效，它可以单独使用或与cache.maxAgeSeconds一起使用。默认值为空。

* `cache.maxAgeSeconds[Number]`：强制规定缓存项目的最大期限（以秒为单位),你可以用这个选项来存储没有自然过期策略的动态资源。例如，可以将cache.maxAgeSeconds设置为例如60 * 60 * 24，这意味着任何超过一天之前的缓存都将被自动删除。此选项仅在同时设置了cache.name时生效，它也可以单独使用或与cache.maxEntries一起使用。默认值为空。




## service worker 容错降级方案

由于 service worker 的持久离线本地缓存的能力，能力越大风险越大。
我们在设计 service worker 的时候需要考虑[容错降级机制](../02-advanced/06-service-worker-compatible.md)。


## 小结

了解上面的这些内容之后，大家可以导出一份工程代码，轻松的完成 Service Worker 调试啦！！！

