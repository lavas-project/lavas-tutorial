# 维护 service worker 文件

servier-worker.js 作为缓存管理的重要文件，在导出工程的时候我们默认给了一个能覆盖缓存需求的 service-worker.js 文件。
但是我们默认提供的文件可能在后续您的开发过程中并不能完全覆盖您的需求，所以你需要对其进行一定的维护。

## service-worker.js

导出项目中，使用了 service worker + sw-precache + sw-precache-webpack-plugin ( Webpack 插件)的方式，在build后自动生成 `service-worker.js` 文件，可满足
* 默认支持离线缓存静态资源能力，通过配置实现动态网络缓存，以及文件更新机制
* 支持 `service-worker.js` 文件更新时，页面自动重载
页面如果没有特殊的缓存需求，可直接使用。为了方便后续工程的定制化，下面会介绍基础使用、开发、维护：



### 如何配置 —— 哪里配置，怎么配置？

开发者可通过`config/sw-precache.js`文件进行缓存配置，根据开发者的缓存配置为用户缓存网站静态与动态资源，并截获用户的所有网络请求，并根据缓存配置来决定是从缓存还是网络获取相应资源，限制缓存大小等。对于无额外需求的开发者，仅需配置该文件就一般可满足项目需求。

具体配置结构如下（更全面的配置可通过[sw-precache](https://www.npmjs.com/package/sw-precache)），该配置在 `webpack.prod.conf.js` 中被 sw-precache-webpack-plugin 作为参数引入，build 时起作用，生成定制化 `service-worker.js` 文件。


看了`config/sw-precache.js`，我们发现还有dev的配置项，这里是为开发者本地调试的提供的，该配置在 `webpack.dev.conf.js` 中被 sw-precache-webpack-dev-server-plugin 作为参数引入，dev-server 时起作用，生成定制化 `service-worker.js` 文件。供调试。


``` js

/* sw-precache.js中的配置 */

build: {

    cacheId: 'my-vue-app',

    /* 生成的文件名称 */
    filename: 'service-worker.js',

    /* 需缓存的文件配置, 可以逐项添加
       需动态缓存的放到runtimeCaching中处理 */
    staticFileGlobs: [],

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



### 如何修改 `service-worker.js` 文件内容 ？？？

如果自动生成的文件实在无法满足项目需求，怎么进行定制化开发呢？ 要想找到答案，我们就要先去看看 sw-precache 工具是怎么生成了这个 `service-worker.js` 文件。

要让 sw-prcahce 工具生成 `service-worker.js` 文件，需要给它提供一个 `.tmpl` 的模板文件。工具默认使用插件默认的模板文件，但是我们也可以通过参考默认模板定制自己的模板，并导入使用，实现定制换开发。通过前面配置文件可以发现，导出的项目中，就是通过 `templateFilePath: 'build/sw.tmpl'` 导入定制化模板来生成`service-worker.js` 文件的， 并且模板文件被提取到了 build 文件夹下，便于开发者后期相应的维护开发。

导出项目中做了什么定制化呢？这就来给大家介绍下，为了在 `service-worker.js` 文件内容更新时，能够让主页面及时做出重载更新，我们在 `build/sw.tmpl` 文件的 `activted` 中通过 `postMessage` 抛出了 `updateMessage` 的信息，在 `sw-register.js` 中，注册了消息的监听，一旦接收到 `updateMessage` 消息，主页面做出 `reload` 的操作重载页面。并在首次注册 service worker 时不发送更新信息，否则会在首次进入页面时，就会再次重载，影响用户体验。


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
                handlerUpdateMessage(e);
            }
        });

        var handleUpdateMessage = function (e) {
            // 在这里可以检测到 service worker 文件的更新，通常我们建议做页面的 reload

            location.reload();
        };
    }
})(window);

```


## 缓存

## 缓存文件数量和大小

## service worker 降级容错方案

## 可能遇到的问题
