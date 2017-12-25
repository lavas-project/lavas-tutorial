# 错误处理

Lavas 的错误处理采用中间件的方式，分别在 express 和 koa 两种模式下添加中间件，用以捕获之后抛出的一切错误，带上错误信息一并跳转到配置好的错误处理路径。因此，除了配置错误路径之外，还需要编写一个错误处理页面，供跳转使用。

## 错误配置

Lavas 的错误处理相关的配置比较简洁，在 `/lavas.config.js` 中找到 `errorHandler` 对象，如下：

```javascript
// ...
errorHandler: {
    errorPath: '/error'
}
// ...
```

唯一的配置项 `errorPath` 用以标识出错时往哪个路由跳转。默认值是 `/error`，因此还需要 `/pages/Error.vue` 存在。如果开发者对默认值能够接受，可以在 `/lavas.config.js` 中省略 `errorHandler` 配置。

## 错误页面

如果采用 `lavas init` 直接生成初始项目，`/pages/Error.vue` 已经存在；如果在上述配置中修改过 `errorPath`，那么在这里我们就需要在 `/pages/` 创建对应的 vue 页面组件。

在错误页面中，我们可以通过 `this.$route.params.error` 来获取错误信息。但这个值也可能不存在，因此还需要设置一个默认值，如：

```javascript
// vue script
computed: {
    message() {
        return this.$route.params.error || 'Oops! Something is not quite right o(╥﹏╥)o';
    }
},
// ...
```

在中间件中，Lavas 会往 `/error?error=xxx` 跳转。如果开发者觉得错误信息直接显示在地址栏无法接受，可以在错误页面再增加一次处理，将错误从 URL 上隐藏起来。

```javascript
// vue script
created() {
    let query = this.$route.query;
    if (Object.keys(query).length !== 0) {
        this.$router.replace({
            name: 'error',
            params: query
        });
    }
}
// ...
```

## 404 错误

由于 Lavas 会自动创建路由规则，因此在创建时对错误路由会进行特殊处理，将错误路径放在最后一条作为备份。如果您对 vue-router 比较熟悉的话，可以参考下面的代码(节选自 `/.lavas/main/route.js`)进行理解：

```javascript
let routes = [
    // other routes ...
    {
        path: '/error',
        name: 'error',
        component: _cb5e100e5a9a3e7f6d1fd97512215282,
        meta: {},
        alias: '*'
    }
];
```

所以当有未被匹配的路由出现时，因为 `alias: *` 的设置，会被匹配到错误路径。这种情况下并没有错误信息，所以错误页面中 `error` 为空，这也是错误页面中 `error` 应当有默认值的原因。

## Lavas 为什么采用跳转而非停留在原页面 (扩展)

*提示：这部分内容由 Lavas 内部处理，并不需要开发者进行参与，仅仅作为解答开发者疑问的扩展阅读存在。*

Lavas 在出错时会携带错误信息往 `/error` (默认情况下) 进行跳转。但是这样会导致用户无法恢复出错前的路由，因为当前的 URL 已经是 `/error`，再次刷新也只会停留在错误页而已。理想的情况应该是 URL 保持出错前用户访问的页面，而页面内容展现为错误页的内容，这样错误恢复时可以通过刷新解决。Lavas 为什么做不到这样呢？

这和 Lavas 内置的 SSR 有关。熟悉 Vue SSR 的开发者应该了解，其中有个步骤叫做客户端激活 ([Client-side Hydration](https://ssr.vuejs.org/zh/hydration.html))，大致是说 Vue 在浏览器端接管由服务端发送的静态 HTML，使其变为由 Vue 管理的动态 DOM 的过程。这里要求前后两端的 HTML 结构是一致的，因此如果出现 URL 和实际渲染的 DOM 不匹配的情况，SSR 会抛出一个错误。

当然从用户体验角度来说，直接跳转的确有损体验，我们会继续学习研究，努力实现更好更顺滑的错误处理模式。
