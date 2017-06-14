# service worker 兼容情况

首先，可以查看 service worker [ 浏览器支持情况](http://caniuse.com/#feat=serviceworkers)

## 浏览器支持情况

可用的浏览器日益增多。服务工作线程受 Firefox 和 Opera 支持。 Microsoft Edge 现在[表示公开支持](https://developer.microsoft.com/en-us/microsoft-edge/platform/status/serviceworker/)。甚至 Safari 也暗示未来会进行相关开发。 您可以在 Jake Archibald 的 is Serviceworker ready 网站上查看[所有浏览器的支持情况](https://jakearchibald.github.io/isserviceworkerready/)。

所以安装服务工作线程，您需要通过在页面中对其进行注册来启动安装。 这将告诉浏览器服务工作线程 JavaScript 文件的位置。