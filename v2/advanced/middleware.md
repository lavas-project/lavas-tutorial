# 中间件

本文将介绍 Lavas 中间件的用法。

如果您想在进入路由组件之前，执行某些统一处理，就可以考虑使用中间件了。在同构应用中，中间件可以运行在客户端，服务端或者两者兼具的场景内。在实现中 Lavas 参考了 [Nuxt](https://zh.nuxtjs.org/guide/routing#中间件) 的实现，通过 vue-router 的路由钩子让中间件顺序执行。

对于开发者，如果想要使用中间件，需要完成以下两步：

1. 将编写的中间件放在项目根目录 `/middlewares` 文件夹下
2. 声明所要使用的中间件名称

下面我们将介绍中间件的具体编写方法。

## 中间件方法说明

编写中间件十分简单，需要创建一个单独文件，放在 `/middlewares` 文件夹下，其中暴露的中间件方法签名如下：
```javascript
// middlewares/my-first-middleware.js

export default function (context) {
    console.log('This is my first middleware.');
}
```

其中中间件上下文对象 `context` 包含以下重要属性方法：

* `isClient|isServer` 当前中间件运行环境，是客户端还是服务端。
* `app` Vue 实例对象。
* `store` vuex 中的 store。
* `route` vue-router 中的[路由对象](https://router.vuejs.org/zh-cn/api/route-object.html)。
* `params` 一个 key/value 对象，包含了 动态片段 和 全匹配片段，如果没有路由参数，就是一个空对象。
* `query` 一个 key/value 对象，表示 URL 查询参数。例如，对于路径 /foo?user=1，则有 $route.query.user == 1，如果没有查询参数，则是个空对象。
* `redirect` 重定向函数。接受唯一的一个参数对象，其中包含如下属性：
    * `status` HTTP 状态码，默认为 302
    * `path` 重定向地址路径
    * `query` 附带参数
* `req` [req 对象](https://expressjs.com/en/api.html#req) 仅限服务端。
* `res` [res 对象](https://expressjs.com/en/api.html#res) 仅限服务端。

利用这个上下文对象，开发者能够实现所需业务逻辑。另外，中间件的实现全都在 Lavas 项目内，其中：

* 上下文对象定义在项目 `core/middleware.js` 文件中的 `getClientContext/getServerContext` 方法内。这意味着开发者可以自由扩展这个上下文对象，挂载任意自定义属性在上面。
* 具体调用这些中间件顺序运行的代码在客户端(`entry-client.js`)和服务端(`entry-server.js`)入口文件中。

## 中间件的声明

中间件编写完毕，我们需要通过配置，声明实际想要使用的中间件。中间件的名称可以声明在以下两个地方：

1. 全局性的中间件，即所有路由组件都会执行的。定义在 `lavas.config.js` 中的 `middleware` 配置项内
2. 路由组件级别的中间件。定义在组件的 `middleware` 属性中

对于全局性的中间件，可以在 `lavas.config.js` 配置文件中配置运行环境：客户端、服务端还是两者兼具。以上面例子中的 `my-first-middleware` 为例，如果想让这个中间件仅在客户端运行，可以这样配置：
```javascript
// lavas.config.js

{
    middleware: {
        all: [], // 同时运行在客户端和服务端
        client: ['my-first-middleware'], // 仅客户端
        server: [] // 仅服务端
    }
}
```

而对于路由组件级别的中间件，只需要定义在组件实例属性中就行了。这样只有这个路由组件会运行这个中间件。如果需要像全局中间件一样指定运行环境，可以通过中间件上下文对象中的 `isClient|isServer` 在中间件内部进行区别处理。
```javascript
// MyComponent.vue
<script>
export default
    {
        name: 'my-component',
        data() {},
        middleware: ['my-first-middleware'],
        // 省略其他属性
    }
</script>
```

注意：Lavas 只会引用用户配置声明的中间件，将其打包在最终构建产物中，并不是 `/middlewares` 下的中间件都会被引用。同样，也不会引用无关的中间件，例如配置仅运行在服务端的中间件及其依赖，并不会被打包进客户端代码中。
