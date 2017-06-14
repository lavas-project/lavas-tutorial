# 维护 service worker 文件

开始之前，您可以查看 [servier worker](https://pwa.baidu.com/doc/offline-and-cache-loading/01-service-worker-introduction) 相关内容，快速掌握相关基础。查看 service worker [ 浏览器支持情况](http://caniuse.com/#feat=serviceworkers)

servier-worker.js 作为缓存管理的重要文件，在导出工程的时候我们默认给了一个能覆盖缓存需求的 service-worker.js 文件。
但是我们默认提供的文件可能在后续您的开发过程中并不能完全覆盖您的需求，所以你需要对其进行一定的维护。

## service-worker.js

导出项目中，使用了 [service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) + [sw-precache](https://github.com/GoogleChrome/sw-precache) + [sw-precache-webpack-plugin](https://www.npmjs.com/package/sw-precache-webpack-plugin)( Webpack 插件)的方式，仅在 build 后自动生成可见的 `service-worker.js` 文件，可
* 支持离线缓存静态资源能力，通过配置实现动态网络缓存，以及文件更新机制
* 支持 `service-worker.js` 文件更新时，页面自动重载。

如果开发者没有特殊的缓存需求，可直接使用。如果开发者需要后续工程的定制化，就要再深入了解以下三方面内容：


## 如何配置缓存内容 —— 哪里配置，怎么配置？

开发者可通过 `config/sw-precache.js` 文件进行缓存配置，根据配置为用户缓存网站静态与动态资源，并截获用户的所有网络请求，决定是从缓存还是网络获取相应资源，限制缓存大小等。对于无额外需求的开发者，一般仅需配置该文件就可满足项目需求。

下面来看一下，具体配置结构（此处给出了一些常用配置，更全面的配置可通过 [sw-precache](https://github.com/GoogleChrome/sw-precache) 查看），该配置在 `webpack.prod.conf.js` 中被 [sw-precache-webpack-plugin](https://www.npmjs.com/package/sw-precache-webpack-plugin) 作为参数引入，build 时起作用，生成定制化 `service-worker.js` 文件。


查看`config/sw-precache.js`文件，我们发现还有dev的配置项，这是为开发者本地调试提供的，该配置在 `webpack.dev.conf.js` 中被 [sw-precache-webpack-dev-server-plugin](https://www.npmjs.com/package/sw-precache-webpack-dev-server-plugin) 作为参数引入，dev-server 时起作用，生成定制化 `service-worker.js` 文件，可在调试窗看到该文件。 在dev-server本地服务和build情况下，缓存的文件路径配置会有一点差别没有`dist/` 路径，调试时大家务必注意。


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
    templateFilePath: 'build/sw.tmpl',

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



## 如何修改 `service-worker.js` 文件内容

**如果自动生成的文件实在无法满足项目需求，怎么进行定制化开发呢？**

要想找到答案，我们就要先去看看 sw-precache 工具是怎么生成了这个 `service-worker.js` 文件。


要让 sw-precahce 工具生成 `service-worker.js` 文件，需要给它提供一个 `.tmpl` 的模板文件。工具默认使用插件默认模板文件，但是我们也可以定制自己的模板（最好参考默认模板），通过配置导入模板，实现定制化开发。在上面的配置文件可以发现，就是通过 `templateFilePath: 'build/sw.tmpl'` 导入定制化模板来生成`service-worker.js` 文件。
**项目中将模板文件提取到了 build 文件夹下，便于开发者后期相应的维护开发。**


**导出项目中做了什么定制化呢？**
这就来给大家介绍下，为了在 `service-worker.js` 文件内容更新时，能够让主页面及时做出重载更新，我们在 `build/sw.tmpl` 文件的 `activate` 监听事件中通过 `postMessage` 抛出了 `updateMessage` 的信息，在 `sw-register.js` 中，注册了消息的监听，一旦接收到 `updateMessage` 消息，主页面做出 `reload` 的操作重载页面。

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


注册部分在项目的 `src/sw-register.js`文件中，并在 `index.html` 引入执行。上面内容提及的 `service-worker.js` 更新时 `updateMessage` 的信息监听和页面重载部分，也是在 `src/sw-register.js`里完成的，开发者可根据需求做相应的扩展。


``` js
// sw-register.js文件, handleUpdateMessage对更新做出重新加载的处理
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
                handleUpdateMessage(e);
            }
        });

        var handleUpdateMessage = function (e) {
            // 在这里可以检测到 service worker 文件的更新，通常我们建议做页面的 reload，开发者也可自定义一些处理

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

缓存内容及策略主要通过 `config/sw-precache.js`配置文件来控制，常用配置的参数如下：
* 配置项中有 `mergeStaticsConfig` 参数（定制化提供参数），默认是 true，即在没配置的情况下，默认缓存所有静态文件
* 如果不想缓存所有的静态文件，需要配置 ` staticFileGlobs` 参数，将需要缓存的静态文件，依次写入
* 对于需要动态缓存的资源，可以通过 runtimeCaching 参数来配置，可指定缓存名称、大小、请求的返回策略（优先网络还是缓存等）


## service worker 容错降级方案

由于 service worker 的持久离线本地缓存的能力，能力越大风险越大。我们在设计 service worker 的时候需要考虑容错降级机制。推荐的做法是采取开关控制模式。这样可以及时全面的控制风险。

```javascript
if (navigator.serviceWorker) {
    fetch(开关的异步接口)
    .then(status => {
        if (status 是 表示降级处理) {
            // 注销所有已安装的 Srevice Worker
        }
        else {
            // 注册 Service Worker
        }
    });
}
```

要注意的有几点：

- 降级一定要注销掉 Service Worker ，而不是简单地不安装。这是因为降级前可能已经有用户访问过网站，导致 Service Worker 被安装，不注销的话降级开关对这部分用户是不起作用的。
- 降级开关需要有即时性，因此服务器和 Service worker 以及浏览器 http 缓存都不应该缓存该接口。
- 降级开关异步接口如果条件允许的话最好走配置配送上线，遇到紧急问题，快速上线才是王道。
- 出现问题并降级后，可能影响问题的排查，因此可以考虑加入对用户隐蔽的 debug 模式（如 url 传入特定字段，debug 模式中忽略降级接口。



## 小结

了解上面的这些内容之后，大家可以导出一份工程代码，轻松的完成 service worker 调试啦！！！

