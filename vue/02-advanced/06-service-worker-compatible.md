# service worker 兼容情况

首先，可以查看 service worker [ 浏览器支持情况](http://caniuse.com/#feat=serviceworkers)

## 浏览器支持情况

可用的浏览器日益增多。服务工作线程受 Firefox 和 Opera 支持。 Microsoft Edge 现在[表示公开支持](https://developer.microsoft.com/en-us/microsoft-edge/platform/status/serviceworker/)。甚至 Safari 也暗示未来会进行相关开发。 您可以在 Jake Archibald 的 is Serviceworker ready 网站上查看[所有浏览器的支持情况](https://jakearchibald.github.io/isserviceworkerready/)。


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
- 降级开关需要有即时性，因此服务器和 Service Worker 以及浏览器 http 缓存都不应该缓存该接口。
- 降级开关异步接口如果条件允许的话最好走配置配送上线，遇到紧急问题，快速上线才是王道。
- 出现问题并降级后，可能影响问题的排查，因此可以考虑加入对用户隐蔽的 debug 模式（如 url 传入特定字段，debug 模式中忽略降级接口。