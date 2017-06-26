# 更新 service worker 缓存

开始之前，您可以查看官网文档 [service worker](https://lavas.baidu.com/doc/offline-and-cache-loading/service-worker/01-service-worker-introduction) 相关内容，快速掌握相关基础。

下文的 service-worker.js 简称 sw.js

## service worker 版本更新

sw.js 控制着页面资源和请求的缓存，那么如果缓存策略需要更新呢？也就是如果 sw.js 有更新怎么办？sw.js 自身该如何更新？

如果 sw.js 内容有更新，当访问网站页面时浏览器获取了新的文件，逐字节比对 sw.js 文件发现不同时它会认为有更新启动[更新算法](https://w3c.github.io/ServiceWorker/#update-algorithm)，于是会安装新的文件并触发 install 事件。但是此时已经处于激活状态的旧的 service worker 还在运行，新的 service worker 完成安装后会进入 waiting 状态。直到所有已打开的页面都关闭，旧的 service worker 自动停止，新的 service worker 才会在接下来重新打开的页面里生效。

#### 如何自动更新所有页面

如果希望在有了新版本时，所有的页面都得到及时自动更新怎么办呢？可以在 install 事件中执行 self.skipWaiting() 方法跳过 waiting 状态，然后会直接进入 activate 阶段。接着在 activate 事件发生时，通过执行 self.clients.claim() 方法，更新所有客户端上的 service worker。

看一下具体实例：

```javascript
// 安装阶段跳过等待，直接进入 active
self.addEventListener('install', function (event) {
    event.waitUntil(self.skipWaiting());
});

self.addEventListener('activate', function (evnet) {
    event.waitUntil(
        Promise.all([

            // 更新客户端
            self.clients.claim(),

            // 清理旧版本
            caches.keys().then(function (cacheList) {
                Promise.all(
                    cacheList.map(function (cacheName) {
                        if (cacheName !== 'my-test-cache-v1') {
                            caches.delete(cacheName);
                        }
                    })
                )
            })
        ])
    );
});
```

另外要注意一点，sw.js 文件可能会因为浏览器缓存问题，当文件有了变化时，浏览器里还是旧的文件。这会导致更新得不到响应。如遇到该问题，可尝试这么做：在 webserver 上添加对该文件的过滤规则，不缓存或设置较短的有效期。


#### 手动更新 sw.js

其实在页面中，也可以手动借助 Registration.update() 更新。

参考如下示例：

```javascript
var version = '1.0.1';

navigator.serviceWorker.register('/sw.js').then(function (reg) {
    if (localStorage.getItem('sw_version') !== version) {
        reg.update().then(function () {
            localStorage.setItem('sw_version', version)
        });
    }
});
```

#### debug时更新

[service worker debug 技巧](./04-service-worker-debug.md)中也会提到, service worker 被载入后立即激活可以保证每次 sw.js 为最新的。

代码如下：

```javascript

self.addEventListener('install', function () {
    self.skipWaiting();
});

```


#### 意外惊喜

 service worker 的特殊之处除了由浏览器触发更新之外，还应用了特殊的缓存策略： 如果该文件已 24 小时没有更新，当 Update 触发时会强制更新。这意味着最坏情况下 service worker 会每天更新一次。

> Set request’s cache mode to “no-cache” if any of the following are true:
> - registration’s use cache is false.
> - job’s force bypass cache flag is set.
> - newestWorker is not null, and registration’s last update check time is not null and the time difference in seconds calculated by the current time minus registration’s last update check time is greater than 86400.



## 缓存更新难题及处理

看完上文的基本的更新机制，我们了解到两个问题：
* 当发布了新版本代码，sw.js 文件本身怎么保证能拿到最新，而不是从缓存中读取。如果 sw.js 不能及时更新，service worker 内的缓存的文件就不能更新，项目中怎么解决呢？
我们在 sw-rigester.js 的请求中增加了一个时间戳（ index.html 中处理），保证每次拿到的sw-register.js 都是最新版本的，且其中 sw.js 文件请求也带有最新的版本参数，保证每次 sw.js 文件请求都与服务器交互，版本参数是在 build/swRegister-webpack-plugin 下完成的。

* 当 sw.js 文件更新后，打开的旧页面并不能及时感知，要重新加载时才能得到更新，这在新版本上线时很容易导致出现问题，所以我们希望在 sw.js 检测到版本更新，重新安装后能够及时的通知主页面（这里不包括首次安装的情况），并做出相应的处理，项目中默认让页面进行 reload 提示处理（src/sw-register.js），大家也可以做进一步的扩展，比如弹层和用户确认，告知用户有新版本，需要重载更新。

上面的两个问题已在导出项目中打包好，开发者可以根据需求来进行扩展开发。









