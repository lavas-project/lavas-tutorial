# 如何使用 Lavas 构建 MPA 项目

Lavas 目前支持两种渲染模式，分别是服务端渲染 (SSR) 和 浏览器端渲染。 而浏览器端渲染时，整个项目构建完成后只有一个 HTML 入口 `index.html`，因此这时本质上等价于单页应用，即 SPA。

从小型站点的开发需求来说，SPA 和 SSR 已经能够覆盖绝大部分。让我们更进一步，考虑如下两种情况：

1. 整个站点的一部分需要使用 SSR 模式渲染，另一部分使用 SPA 模式渲染。

2. 整个站点均为浏览器端渲染，但存在多个入口 HTML，即多页应用 (MPA)。

如何使用 Lavas 来满足此类需求呢？

## 解决思路

我们可以把 Lavas 项目看做一个单独的单元，或称入口。

对于上述两种情况，都可以认为由多个入口组成：

1. 情况 1，相当于一些入口采用 SSR 模式，另一些入口采用 SPA 模式

2. 情况 2，相当于项目中存在多个 SPA 模式的入口

因此，我们把问题归结为：多个 Lavas 项目如何整合到一起？

在 Lavas 构建完成后，SPA 项目本质上是一些静态页面，包含一个入口 `index.html` 和其他静态资源；SSR 项目本质上是一个 express (也可以是 koa )中间件。因此通过架设一个 express 服务器可以很方便分别对两者进行整合。

## 实现方式

为了表述方便，我们来创建一个简单但可能很实用的需求，通过解决这个需求来学习如何整合。

### 示例需求

我们假设开发者存在这样的开发需求：一个电商站点从业务模块角度能分成两部分，分别是：

* `/user/*` 部分，用户信息浏览/注册/修改等等相关，采用 SPA 模式渲染。一般这类用户信息敏感的内容不需要考虑 SEO，也就不需要 SSR。

* 剩余部分，是站点的主要内容，例如 `/` 展示站点首页，`/detail/*` 查看商品详情，`/search/*` 进行商品搜索等等。这部分考虑到 SEO 需求，使用 SSR 模式进行渲染。

我们假设开发者已经分别为两者开发了单独的 Lavas 应用。前者名为 lavas-user，后者名为 lavas-main。我们需要这么几个步骤：

1. 修改基础路由互相区分

2. 提取共享模块避免重复 (可选)

3. 配置服务器

4. 构建

### 修改基础路由

观察示例需求的两个模块，lavas-user 拥有明显的 URL 特征 ( `/user` 开头)，而 lavas-main 没有。因此我们修改 lavas-user 的 `base`，lavas-main 不作修改。

打开 `lavas-user/lavas.config.js` 配置文件的`build` 段的 `publicPath` 以及 `router` 段的 `base`，均修改为 `/user/` (__不要遗漏最后的 `/` ！__)，如下：

```javascript
// ...
build: {
    // ...
    publicPath: '/user/'
},
router: {
    mode: 'history',
    base: '/user/',
    pageTransition: {
        enable: false
    }
}
// ...
```

>info
>`base` 配置项是 vue-router 的一个配置项，用以设定基准路由。修改后的 lavas-main，原本使用 `/view` 的路由就变成了 `/user/view`， 原本使用 `/register` 就变成了 `/user/register`，以此类推。
>为了配合 `base`， 用以管理静态资源路径前缀的配置项 `publicPath` 也应做相同的修改，否则会导致系统找不到静态资源而报错。

### 提取共享模块 (可选)

如果 lavas-main 和 lavas-user 两个模块都引用了相同的内容，开发者又对重复代码无法接受，可以考虑将共同的代码抽离出来。

举例来说，由 `lavas init` 初始化的项目都会有 `/components` 目录，里面会有共同使用的组件 (离线通知，Service Worker 更新通知和页面切换进度条)。如果要把这块提取出来的话，我们可以通过 webpack 的 `alias` 实现。

```
lavas-project
├── components/
│   ├──OfflineToast.vue
│   ├──UpdateToast.vue
│   └──ProgressBar.vue
├── lavas-user/
│   └──lavas.config.js
└── lavas-main/
    └──lavas.config.js
```

Lavas 为开发者提供了 `alias` 配置项([文档](/guide/v2/advanced/build-config#alias))，修改 `/lavas.config.js` 即可。

```javascript
// ...
build: {
    alias: {
        base: {
            '@': path.resolve(__dirname, '../')
        }
    }
}
```

在实际引用时 (如 `App.vue` 中)，采用的引用路径是 `@/components/xxx.vue`，因此将 `'@'` 路径配置成 `path.resolve(__dirname, '../')` 可以避免修改这些引用处，比较方便。

### 配置服务器

最后一步是在两个 Lavas 服务之前搭建一个分发服务器，对不同的 URL 转发到不同的 Lavas 服务。以 express 为例，我们新建 `server.js`，内容如下：

```javascript
const path = require('path');
const express = require('express');
const app = express();
const historyMiddleware = require('connect-history-api-fallback');
const LavasCore = require('lavas-core-vue');
const port = 8080; // 对外端口

function registerSPA(url, dirPath) {
    if (url.endsWith('/')) {
        url = url.substring(0, url.length - 1);
    }

    // fix trailing slash (/user -> /user/)
    app.use('/', function (req, res, next) {
        let requestUrl = req.url.replace(/\?.+?$/, '');

        if (requestUrl === url) {
            req.url = url + '/';
        }

        next();
    });

    app.use(url, historyMiddleware({
        htmlAcceptHeaders: ['text/html'],
        disableDotRule: false // ignore paths with dot inside
    }));


    app.use(url, express.static(dirPath));
}

// SPA
registerSPA('/user', 'lavas-user/dist');

// NOT required when SSR is enabled
// app.listen(port, () => {
//     console.log('server started at localhost:' + port);
// });

// SSR
let core = new LavasCore(path.resolve(__dirname, 'lavas-main/dist'));

core.init('production')
    .then(() => core.runAfterBuild())
    .then(() => {
        app.use(core.expressMiddleware());
        app.listen(port, () => {
            console.log('server started at localhost:' + port);
        });
    }).catch(err => {
        console.log(err);
    });

// catch promise error
process.on('unhandledRejection', (err, promise) => {
    console.log('in unhandledRejection');
    console.log(err);
    // cannot redirect without ctx!
});

```

文件中的所有项目文件路径(如 `lavas-user/dist`)以及启动端口号(如 `8080`)都可以根据项目实际情况进行修改。

文件的上半部分对 SPA 进行处理，核心是把 `/user` 开头的路由转发到 lavas-user 的入口 `lavas-user/dist/index.html` 上。其中还涉及到一个 URL 结尾 `/` 的小问题，我们在[最后](/guide/v2/advanced/multi-lavas#express-%E5%A4%84%E7%90%86-spa-%E8%B7%AF%E7%94%B1%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98-%E6%89%A9%E5%B1%95)进行叙述。SPA 部分的最后是启动 express 服务器并监听端口，但因为 SSR 部分包含异步操作，因此 __如果项目包含 SSR 部分，则这里可以注释，由 SSR 部分负责启动__。

文件后半部分是对 SSR 进行处理，这部分和 `lavas-main/server.prod.js` 比较类似，就不再赘述了。

### 构建

1. 分别对两个项目使用 `lavas build` 命令进行构建

2. 复制任意一个项目的 `node_modules` 目录到根目录，供上述 `server.js` 使用

3. 如果想精简项目内容，可以只将两个项目的 `dist` 目录移动出来，其余源码部分和 `node_modules` 都可以去除 __(可选)__

### 最终目录结构

如果按照文档上列出的 `server.js` 中的配置，最终目录应该如下：

```
lavas-project
├── lavas-user/
│   ├── dist
│   │   ├── index.html
│   │   └── something else (favicon.ico, lavas/, static/...)
│   └── something else (node_modules/, .lavas/, pages/, store/...)
│
├── lavas-main/
│   ├── dist
│   │   ├── server.prod.js
│   │   └── something else (lavas/, node_modules/, static/...)
│   └── something else (node_modules/, .lavas/, pages/, store/...)
│
├── node_modules/
└── server.js
```

更进一步，精简后的代码结构可以是：

```
lavas-project
├── lavas-user-dist/
│   ├── index.html
│   └── something else (favicon.ico, lavas/, static/...)
│
├── lavas-main-dist/
│   ├── server.prod.js
│   └── something else (lavas/, node_modules/, static/...)
│
├── node_modules/
└── server.js
```

这应该是上线需求的最小集合了，为了适应这样的修改，还需要对 `server.js` 中的引用路径进行改动，这里就不重复了。

### express 处理 SPA 路由的小问题 (扩展)

*提示：这部分内容由 Lavas 内部处理，并不需要开发者进行参与，仅仅作为解答开发者疑问的扩展阅读存在。*

在 `server.js` 中，我们能够发现存在一段代码：

```javascript
// fix trailing slash (/user -> /user/)
app.use('/', function (req, res, next) {
    let requestUrl = req.url.replace(/\?.+?$/, '');

    if (requestUrl === url) {
        req.url = url + '/';
    }

    next();
});
```

这段代码是用来处理一个 express 的路由问题的。Vue 官方推荐开发者在上线 Vue SPA 项目时使用 [connect-history-api-fallback](https://github.com/bripkens/connect-history-api-fallback)，这个中间件的核心是修改 express 的 `req.url`，让我们看看如下代码(截取自该中间件)：

```javascript
rewriteTarget = options.index || '/index.html';
logger('Rewriting', req.method, req.url, 'to', rewriteTarget);
req.url = rewriteTarget;
next();
```

然后我们使用方式如下：

```javascript
app.use('/user', historyMiddleware({
    htmlAcceptHeaders: ['text/html'],
    disableDotRule: false // ignore paths with dot inside
}))
```

在这种配置下，当我们访问 `/user/`，经过中间件之后 `req.url` 会被设置为 `/user/index.html`，再进入 `express.static`，一切正常。但当访问 `/user` 时(没有后面那个 `/`)，经过中间件之后会变成 `/userindex.html`，这样是无法被 `express.static` 识别的，当然落到 SSR 之后也无法匹配，因此会报出 404 错误。

因此我们在使用中间件之前还增加了一段修复代码，在访问 `/user` 的时候自动添加最后的 `/`。我们也考虑过 express 的 strict routing，但似乎也没法生效。如果开发者有更好的方法，欢迎告诉我们！
