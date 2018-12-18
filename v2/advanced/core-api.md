# 以编程方式使用 Lavas

使用 `lavas init` 创建的模板项目中，在以下场景下都会以编程方式使用 Lavas：
* `server.dev.js` 开发环境下的 SPA／SSR 模式。
* `server.prod.js` 生产环境下的 SSR 模式。

可见以编程方式使用 Lavas 的主要场景就是 SSR 模式，而在 SPA 模式下仅仅是供开发服务器使用。因此，如果开发者选择了 SSR 模式，阅读下面的内容将十分有帮助：
* 如何选择性使用 Lavas 内置中间件
* SSR 场景下使用 `render()` 方法自定义服务端渲染中间件

## Lavas 内置中间件

Lavas 封装了一系列中间件，暴露 `express/koaMiddleware()` 方法供开发者便捷使用。对于简单的应用场景，直接调用即可运行：
```javascript
// 以 express 为例
app.use(core.expressMiddleware());
```

下面我们将详细介绍 Lavas 内置的中间件，以便开发者了解内部细节之后进行某些中间件的替换。
- `static` 响应静态资源
    - express 使用了 [serve-static](https://github.com/expressjs/serve-static)
    - Koa 使用了 [koa-static](https://github.com/koajs/static)
- `service-worker` 响应 `sw-register.js` 和 `service-worker.js`。这两个文件需要禁止缓存，具体原因可见[这篇文章](https://zhuanlan.zhihu.com/p/28161855)。如果您需要自己实现响应这两个文件的中间件，可以参考 Lavas 的内部实现：
    - express 使用了 [serve-static](https://github.com/expressjs/serve-static)。这个中间件默认设置了 `etag: true` 进行浏览器缓存，我们需要关闭这一默认选项
    - Koa 手动设置了响应头 `'Cache-Control', 'private, no-cache, no-store'`
- `ssr` 使用创建好的 `bundleRenderer` 渲染请求对应的 HTML 内容。稍后将介绍如何使用开发者自定义的中间件进行替换
- `error` 处理错误，内部会跳转到配置的错误路由路径，详见[错误处理](/guide/v2/advanced/error-handler)
- `compression` 压缩响应体，使用了 [compression](https://github.com/expressjs/compression)中间件
- `favicon` 响应 `favicon`，使用了 [serve-favicon](https://github.com/expressjs/serve-favicon)中间件

对于这些中间件，开发者可以选择性使用。例如不想使用 `static` 中间件处理静态资源，而采用 nginx 处理，可以从中间件列表中去掉 `static` 这个中间件：
```javascript
// 去掉 static 中间件
app.use(core.koaMiddleware([
    'favicon',
    // 'static',
    'service-worker',
    'ssr',
    'error'
]));
```

> info
>
> 在开发模式下，Lavas 使用了 `webpack-dev-middleware` 和 `webpack-hot-middleware` 从内存文件系统响应静态资源以及实现 hotreload 相关功能，对于这两个通用的中间件，我们并不打算提供关闭功能。

## 使用 `render()` 自定义 SSR 中间件

为了满足对于自定义 SSR 中间件的需求，Lavas 提供了 `render()` 方法。以 express 为例，可以这样定义一个简单的 SSR 中间件：
```javascript
// 自定义 SSR express 中间件
app.use((req, res, next) => {
    // render 方法
    core.render({
        url: req.url
    }).then(result => {
        if (result.err) {
            return next(result.err);
        }
        res.end(result.html);
    });
});
```

`render()` 方法接收的参数对象包含如下属性：
* `url` 请求 URL，供 vue-router 查找匹配的路由对象
* `req` 请求对象，供后续服务端中间件访问
* `res` 响应对象，供后续服务端中间件访问

而方法返回的 Promise 对象包含以下两个属性：
* `err` 渲染过程中的错误对象
* `html` HTML 结果内容

### 注意事项

Lavas 内置的中间件以固定的顺序执行，这是由于某些中间件的执行顺序是不能随意更改的，例如错误处理必须放在最后。

因此，在以下场景下，例如开发者如果只想替换 SSR 中间件，而仍旧使用 Lavas 内置的错误处理中间件，可以这样：
```javascript
// 其余 Lavas 中间件
app.use(core.expressMiddleware(['static', 'service-worker']));
// 自定义 SSR express 中间件
app.use((req, res, next) => {});
// Lavas error 中间件必须放到最后
app.use(core.expressMiddleware('error'));
```
