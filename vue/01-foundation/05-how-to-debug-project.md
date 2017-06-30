# 调试工程

## webpack dev-server

> webpack-dev-server 是一个小型的 Node.js Express 服务器,它使用 webpack-dev-middleware 中间件来为通过 Webpack 打包生成的资源文件提供 Web 服务。它还有一个通过 Socket IO 连接着 webpack-dev-server 服务器的小型运行时程序。webpack-dev-server 发送关于编译状态的消息到客户端，客户端根据消息作出响应。

我们的导出工程中默认会集成安装 Webpack dev-server, 主要用途是用于开发和调试 Vue 工程。

详细了解 [Webpack dev-server](http://webpack.github.io/docs/webpack-dev-server.html)。

我们的工程中，通过执行以下命令启动 dev-server

```npm
$ npm run dev
```

这个时候，如果代码没有问题，我们可以看到 server 启动成功的控制台信息。

![dev-server success](./images/dev-server.png)

并且会自动在默认浏览器中打开页面，载入我们的工程运行后的页面效果。然后在开发过程中，dev-server 通过监听代码文件变化时页面实现热重载，动态编译和热重载到打开的页面。这样就做到所写即所得的开发体验。调试起来非常方便。

当然，如果代码中出现问题的话。在浏览器和控制台都会有相应的信息打印出来以供排查问题。


## chrome 调试

我们在 Chrome 上调试分为两个部分

**常规页面调试**
Chrome DevTools 提供了强大的调试能力，网络，资源，错误监控，缓存等调试都可以在 Chrome DevTools 进行绝大部分的调试。可以在初期的时候帮助我们避免掉很多问题。详见 [Chrome DevTools 官方文档](https://developers.google.cn/web/tools/chrome-devtools/)。

**service worker 调试**
我们可以通过 Chrome DevTools 进行 service worker 的调试。具体做法请看 [service worker 调试](https://lavas.baidu.com/doc/offline-and-cache-loading/service-worker/04-service-worker-debug)

