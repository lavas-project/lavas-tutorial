# 路由配置项

路由配置项位于 `/lavas.config.js` 的 `router` 对象。Lavas 内部使用 vue-router 进行路由管理，因此许多配置项都和 vue-router 是相同的。

Lavas 路由配置项包括：

* 路由模式，基准路由等
* 路由切换动画效果
* 重写路由(如果对自动生成的路由规则不满意)

## 路由模式和基准路由

```javascript
router: {
    mode: 'history',
    base: '/',
    // ...
}
```

`mode` 和 `base` 都是 vue-router 的配置项，分别用来定义路由模式和路由基础路径。

`mode` 可选值有 `'hash'` 和 `'history'` (`'abstract'` 只作用于服务端，这里并没有使用的必要)。注意的是在 SSR 模式下__不支持__ `hash`。

`base` 用以定义整个项目的的基础路径，正常情况为 `/`。如果开发者希望将整个服务部署在 `https://some.domain/app/`， 那么这里就应该填写 `/app/`。注意最后的 `/` __不能遗漏__。

这两个配置项也可以参考 vue-router 的文档中 [mode](https://router.vuejs.org/zh-cn/api/options.html#mode) 和 [base](https://router.vuejs.org/zh-cn/api/options.html#base) 部分。

## pageTransition

```javascript
router: {
    pageTransition: {
        type: 'fade',
        transitionClass: 'fade'
    },
    // ...
}
```

 TODO

## 重写路由对象

Lavas 会根据 `/pages` 文件夹内的目录结构，自动生成 vue-router 路由对象。具体生成规则可以参考[这里](/guide/v2/basic/init#Lavas-自动路由生成方法)。

在某些复杂场景下，如果自动生成不能够完全满足需求，此时就需要通过路由配置对象进行重写了。

```javascript
router: {
    rewrite: [],
    routes: [],
    // ...
}
```

下面我们将分别介绍这两个配置项的用法和使用场景。

### 使用 rewrite 修改路由路径

这个配置项可以定义一组规则，用来重写 Lavas 自动生成的路由路径。举例来说，如果项目中存在 `/pages/Detail.vue` 这样一个路由组件，Lavas 会自动生成 `/detail` 这样一条路由路径，如果想重写这条路径，例如加上 `rewrite` 前缀，可以这么做：
```javascript
rewrite: [
    {
        from: '/detail',
        to: '/rewrite/detail'
    }
]
```

其中每一条重写规则包括两个属性：
* `from` 匹配规则，支持以下三种类型：
    1. 正则 `RegExp`，内部会调用 `path.replace(from, to)` 进行路由路径的重写
    2. 数组 `Array`，存在于数组中的路由路径会被重写
    3. 字符串，完全匹配的路由路径会被重写
* `to` 重写路径，`from` 为数组或字符串时，必须是字符串类型；`from` 为正则时，除了字符串还可以是一个函数，可以参考[ replace 指定一个函数作为参数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace#指定一个函数作为参数)

#### 注意事项

这里有一点需要注意，在 `rewrite` 中与 `from` 进行匹配的路由路径，是 `route.path`，并不是路径的完整形式，即 `route.fullPath`。

请看下面这个嵌套路由的例子，包含了一个父路由 `/detail` 和一个子路由 `/detail/:id`：
```javascript
{
    path: '/detail',
    name: 'detail',
    component: _1513...,
    children: [{
        path: ':id',
        name: 'detailId',
        component: _15130...
    }]
}
```

应用以下 `rewrite` 规则将影响父路由组件的路径，当然子路由路径也跟着变成了 `/rewrite/detail/:id`：
```javascript
rewrite: [
    {
        from: '/detail',
        to: '/rewrite/detail'
    }
]
```

在上面的场景中一切正常，但如果仅仅想修改子路由组件，而不想影响到父路由组件，我们的 `rewrite` 规则就只能这么写了，类似 `':id'` 这样的写法实在很容易造成重名现象，进而影响到其他不相干的路由：
```javascript
rewrite: [
    {
        from: ':id',
        // 不能写成 /detail/:id
        to: '/rewrite/:id'
    }
]
```

可以看出 `rewrite` 适合重写父组件的路径，从而批量影响其下的所有子路由。而在需要精确修改某一条路由尤其是子路由的时候，就需要使用下面介绍的 `routes` 了。

### 使用 routes 修改路由对象

虽然用 `rewrite` 可以便捷地重写路由路径，但是路由对象不仅仅只有路径这一个属性。如果想更精确地修改 Lavas 自动生成的路由对象，可以使用 `routes` 配置项。该配置项是一个包含了路由对象的数组，其中路由对象包含以下可配置项：

* `pattern` 字符串或者是正则。与 `rewrite` 不同，使用 `route.fullPath` 进行匹配
* `path` 重写路由路径。
* `lazyLoading` 是否需要[路由懒加载](https://router.vuejs.org/zh-cn/advanced/lazy-loading.html)，布尔值。
* `chunkname` 异步路由 chunk 名称。最终会生成一个 `[chunkname].[hash].js` 异步路由文件。配置此项后可以不用额外配置 `lazyLoading`，默认打开
* `alias` [别名](https://router.vuejs.org/zh-cn/essentials/redirect-and-alias.html)，在这里也可以进行路由路径的重新定义，但是由于仅仅是别名，原路由路径依然可以访问
* `meta` [路由元信息](https://router.vuejs.org/zh-cn/advanced/meta.html)，可以将一些自定义属性挂载在上面，便于后续访问
* `scrollBehavior` [滚动行为](https://router.vuejs.org/zh-cn/advanced/scroll-behavior.html)函数。如果不配置，将使用内置的默认函数，切换路由时将记录上一次的滚动距离，对于希望滚动到顶的路由组件，可以在路由元信息中配置 `meta.scrollToTop` 为 `true`

在下面的例子中，我们希望将所有路径中包含 `/detail` 的路由组件打包到一个 chunk 中，并命名为 `my-chunk`，同时加上 `keepAlive` 这个元信息供后续使用：
```javascript
routes: [
    {
        pattern: /\/detail/,
        lazyLoading: true,
        chunkname: 'my-chunk',
        meta: {
            keepAlive: true
        }
    }
]
```
