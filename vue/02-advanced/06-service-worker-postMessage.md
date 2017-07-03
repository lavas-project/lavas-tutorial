# Service Worker 与页面通信

[service worker](https://developers.google.com/web/fundamentals/getting-started/primers/service-workers?hl=zh-cn) 没有直接操作页面 DOM 的权限，但是可以通过 postMessage 方法和 Web 页面进行通信，让页面操作 DOM。而且这种通信可以是双向的，类似于 iframe 之间的通信。下面就给大家介绍 postMessage 在项目中的一些使用场景。注意下面的前提是浏览器支持 service worker。

下文的 `service-worker.js` 文件，简称为 `sw.js`。


## 如何使用 postMessage 方法发送信息 ？

1、在 `sw.js` 中向接管页面发信息，可以采用 client.postMessage() 方法，示例代码如下：

```js
self.clients.matchAll()
    .then(function (clients) {
        if (clients && clients.length) {
            clients.forEach(function (client) {
                // 发送字符串'sw.update'
                client.postMessage('sw.update');
            })
        }
    })
```


2、在主页面给 service worker 发消息，可以采用 navigator.serviceWorker.controller.postMessage() 方法，示例代码如下：

```js
// 点击指定 DOM 时就给service worker 发送消息
document.getElementById('app-refresh').addEventListener('click', () => {
    navigator.serviceWorker.controller && navigator.serviceWorker.controller.postMessage('sw.updatedone');
});
```


## 如何接收 postMessage 发送的信息 ？

若要接收消息，当然我们需要绑定 message 的监听事件

1、在 `sw.js` 中接收主页面发来的信息，示例代码如下，通过 event.data 来读取数据：

```js
self.addEventListener('message', function (event) {
    console.log(event.data); // 输出：'sw.updatedone'
});
```

2、在页面中接收 `sw.js` 发来的信息，示例代码如下，通过 event.data 来读取数据：

```js
navigator.serviceWorker.addEventListener('message', event => {
    if (e.data === 'sw.update') {
        // 此处可以操作页面的 DOM 元素啦
    }
});
```


## 实现 sw.js 的检测更新机制

我们利用这种通信，为您在导出项目中做了一些简单的 `sw.js` 缓存更新，在上一节中的[缓存更新及处理](./05-service-worker-maintenance#缓存更新难题及处理)中有相应的阐述，这里具体展开一些实现，以及您后期可进行的升级

`sw.js` 文件发现更新后，在 activate 事件最后 postMessage 事件（代码在导出项目中的 `sw.tmpl.js` 文件）

```js
self.addEventListener('activate', function (event) {

    event.waitUntil(
        caches.open(cacheName).then(function (cache) {
            // 省略
        }).then(function() {

            // 如果非首次安装 service worker 或缓存中原先有缓存的静态资源，我们需要通知接管页面，sw.js有更新，提示用户点击刷新页面
            if (!firstRegister) {
                return self.clients.matchAll()
                    .then(function (clients) {
                        if (clients && clients.length) {
                            clients.forEach(function (client) {
                                client.postMessage('sw.update');
                            })
                        }
                    })
            }
        })
    );
});
```

在页面中，接收到消息 service worker 的缓存更新消息后，在页面增加提示，如下图所示（代码在导出项目中的 `src/sw-register.js` 文件）

```js
// src/sw-register.js 中注册，重载相关代码
navigator.serviceWorker && navigator.serviceWorker.register('/service-worker.js').then(() => {
    // 监听 message 事件，并处理
    navigator.serviceWorker.addEventListener('message', e => {

        // service worker 如果更新成功会 postMessage 给页面，内容为 'sw.update'
        if (e.data === 'sw.update') {

            // 开发者可以使用默认提供的用户提示，引导用户刷新
            // 也可以自定义处理函数，这里建议引导用户 reload 处理，详细查看项目具体文件
            // location.reload();
        }
    });
});
```
![版本更新提示引导](./images/refreshTip.png)


## 可能存在的问题及解决方案

**1、问题**

结合上面的更新提示机制，当我们在浏览器中打开多个相同的页面时，若 `sw.js` 文件更新成功，多个窗口均会弹出引导用户更新的提示条，当用户点击当前页面的 "点击刷新" 时，我们会重载当前页面，当切换至其他页面时，提示条仍然可见，并且还需用户点击刷新才能更新老页面，如果不刷新就可能存在老页面使用新缓存的问题，在新版本上线时，有一定的风险。使用者可以根据情况选择是否要做进一步升级。

**2、解决方法**

解决上述问题的方法也并不复杂，需要利用浏览器的 visibilitychange 事件，这是浏览器新添加的一个事件，当浏览器的某个标签页切换到后台，或从后台切换到前台时就会触发该消息，现在主流的浏览器都支持该事件了，例如 Chrome, Firefox, IE10 等。当切换某个页面时，就会触发该事件，可通过判断 “有刷新提示条 & 用户点击刷新” 来决定是否重载页面。


```js

// 可以监听的事件名称
var visibilityChangeEvent = '';
if (document.hidden) {
    hiddenProperty = 'visibilitychange';
}
if (document.wekitHidden) {
    hiddenProperty = 'wekitvisibilitychange';
}
else if (document.mozHidden) {
    hiddenProperty = 'mozvisibilitychange';
}

// 如果支持该事件，就绑定并添加处理函数
if (visibilityChangeEvent) {
    var onVisibilityChange = function(){
        // 在进入页面和离开页面均会触发该事件，所以我们这里需要判断是进入页面的情况才做处理
        if (!(document.hidden || document.wekitHidden || document.mozHidden)) {
            // 刷新提示条显示类名，判断是否有刷新条，这里只是示例
            var hasRefreshTip = document.getElementsByClassName('app-refresh-show').length;
            // 刷新提示是否被用户点击刷新，这里仅示例
            var userClickRefresh = document.getElementsByClassName('app-refresh-click').length;

            // 有刷新提示条 && 某个页面点击过刷新
            if (hasRefreshTip && userClickRefresh) {

                // 这里的location.reload只能刷新当前打开的页面，后台的页面并不起作用
                location.reload();
            }
        }
    }
    document.addEventListener(visibilityChangeEvent, onVisibilityChange);
}
```

通过上面示例代码我们看到若要刷新页面，还需要关键的一个判断条件 —— 某个页面用户已点击过刷新。如果其他页面没点击过刷新，我们也不应该重载更新。其实，这里使用“双向通信”就可以解决这个问题了。当某个页面用户点击刷新后，给 service worker 发送一个 "sw.updatedone" 的消息，service worker 接收到该消息以后，可以给接管的后台页面发送该消息，后台页面接收到信息后，可以对 DOM 做相应的操作，如给特定标签增加一个指定类名等，用于页面激活后判断用户是否已点击过刷新。这样就简单的解决了

注意，不能在后台页面接收到 "sw.updatedone" 消息后就直接 reload, 会不起作用，浏览器只能 reload 当前的页面。

## 扩展

查看资料时，发现 [Polymer 示例](https://github.com/StartPolymer/progressive-web-app-template)使用了 [MessageChannel](https://developer.mozilla.org/zh-CN/docs/Web/API/MessageChannel) 的方式 postMessage，大家也可以了解下。

MessageChannel 接口是信道通信API的一个接口，它允许我们创建一个新的信道并通过信道的两个 MessagePort 属性来传递数据

简单来说，MessageChannel 创建了一个通信的管道，这个管道有两个口子，每个口子都可以通过 postMessage 发送数据，而一个口子只要绑定了 onmessage 回调方法，就可以接收从另一个口子传过来的数据。

一个简单的例子：
```js

var ch = new MessageChannel();
var p1 = ch.port1;
var p2 = ch.port2;
p1.onmessage = function(e){console.log("port1 receive " + e.data)}
p2.onmessage = function(e){console.log("port2 receive " + e.data)}
p1.postMessage("你好世界");
p2.postMessage("世界你好");

// 输出
// port1 receive 世界你好
// port2 receive 你好世界

```

MessageChannel 用法很简单，但是功能却不可小觑。例如当我们使用多个 web worker 之间实现通信的时候，MessageChannel 就可以派上用场。

## 小结

本文主要介绍所需的一些基础知识和示例，开发者可以根据自身项目需求进行相应的定制。










