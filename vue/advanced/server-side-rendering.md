服务器端渲染
============

Lavas 服务器端渲染模板参考了 [vue-hackernews](https://github.com/vuejs/vue-hackernews-2.0) 的渲染和开发机制，并且结合了 Lavas 的 App Shell 模板，导出的工程中会有 App Shell 等

如果您不了解 [vue 的服务器端渲染](https://ssr.vuejs.org/zh/)，没关系，这就是 Lavas 存在的意义，让您不需要过多的关心 vue 的实现机制。

Lavas 服务器端渲染模板除了服务器端渲染的机制和 vue 本身的代码之外，还包括以下：

- App Shell + 组件库 [vuetify](https://vuetifyjs.com/vuetify/quick-start)
- [图标解决方案](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/03-how-to-use-icon)
- [主题解决方案](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/04-how-to-change-theme)，和 vuetify 绑定
- [service worker 解决方案](https://lavas.baidu.com/guide/vue/doc/vue/02-advanced/05-service-worker-maintenance)

如果不做一些定制化的修改，开发者基本上不需要关心上面这些问题。

下面我会用几个章节来讲如何使用 Lavas 搭建一套服务器端渲染的以及剖析一下 vue 的服务器端渲染机制。

## 使用 Lavas 创建服务器端渲染项目

### 导出项目

Lavas 提供的 SSR 模板导出通过

- 安装 lavas 的命令行工具，`npm install -g lavas`
- 通过 `lavas init ` 创建项目
- 选择 Server Side Rendering 模板，输入基本信息确认导出

```bash
$ lavas init

? 选择一个模版类型 (按上下键选择):
  Basic
    简易单页应用模版，包含 PWA 工程化相关必需内容。
  App Shell
    AppShell 模版，其中包含 PWA 工程化必需内容以及通用 shell 的封装。
  Multiple Page App (多页应用)
    多页应用模版，其中包含多页应用工程化解决方案以及 PWA 工程化必需内容。
❯ Server Side Rendering (服务端渲染)
    SSR PWA 模版，其中包含 SSR 工程化解决方案以及 PWA 工程化必需内容
```

完成上面的步骤之后，会在当前目录生成项目目录，其中有几个文件需要和 SSR 开发调试强相关，需要大家关注

	./
		| - server.js （development 和 production 的启动入口文件）
		| - build/ （Webpack 调试和构建文件目录）
			| - setup-dev-server.js （调试环境下启动的调试服务器）
			| - webpack.server.conf.js （服务器端渲染的 Webpack 配置文件）
		| - src/
			| - entry-server.js （服务器端渲染的入口文件）
			| - index.template.html （服务器端渲染的 layout）


### 启动

- 通过 `npm install` 安装依赖
- 通过 `npm run dev` 启动

启动完成之后，通过 `http://localhost:8080` 访问即可预览，查看页面源代码可以看到返回的内容里面是已经渲染好的结果。

至此，Lavas 的目的已经达到了，让开发者无成本的创建一个服务端渲染的项目。

接下来我们对这套 SSR 模板进行一下解析，看如何运行起来的。

## 深度剖析 SSR

这个章节，我们了解一下 SSR 模板如何运作的。

首先，来看一下入口文件[ `server.js`](https://github.com/lavas-project/lavas-template-vue-ssr/blob/master/server.js)，在这个文件中，通过 express 启动了一个服务器，在这个文件中，是通过 `renderer.renderToString` 来完成渲染的，我们可以看看这个 `renderer` 是怎么生成的

```javascript
if (isProd) {
	// 在生产环境中，通过 vue-ssr-server-bundle.json 文件创建 renderer
	// 这个文件是通过 vue-ssr-webpack-plugin 生成的
    const bundle = require('./dist/vue-ssr-server-bundle.json');

	// vue-ssr-client-manifest.json 文件是可选的
    const clientManifest = require('./dist/vue-ssr-client-manifest.json');

    renderer = createRenderer(bundle, {
        clientManifest
    });
}
else {
	// 开发环境中，setup-dev-server 会热重载 vue-ssr-server-bundle.json 文件，所以，需要重新创建 renderer
    readyPromise = require('./build/setup-dev-server')(app, (bundle, options) => {
        renderer = createRenderer(bundle, options);
    });
}
```

这里区分了生产环境和开发环境，在开发环境中，会实时编译生成 `vue-ssr-server-bundle.json` 文件。

熟悉 vue 的 SSR 的开发者知道，vue SSR 依赖的文件有三个

- `vue-ssr-server-bundle.json` - 包含打包好的所有代码
- `vue-ssr-client-manifest.json` - 可选，静态文件清单
- `index.template.html`  - 可选，模板文件，这个文件内容是固定的

接下来我们看看这两个 JSON 文件是怎么生成的

### 如何生成 vue-ssr-server-bundle.json

`vue-ssr-server-bundle.json` 文件是通过 [`build/webpack.server.conf.js`](https://github.com/lavas-project/lavas-template-vue-ssr/blob/master/build/webpack.server.conf.js) 经过 webpack 编译后文件生成的。

在这个文件中，我们指定的 entry 是 `entry-server.js`文件，而不是 `entry-client.js`，这两个文件虽然运行在不同的环境之中的，但是做的事情却很相似：获取当前匹配的组件，调用组件的 `asyncData` 方法，在 `asyncData` 方法中，请求异步数据，设置 `state`，最后再将 `store.state` 赋值给 `context`，服务器端渲染之后，输出到页面中。

关于服务器端渲染中的数据预取，请看 [vue 官方提供的文档](https://ssr.vuejs.org/zh/data.html)。


### 如何生成 vue-ssr-client-manifest.json

`vue-ssr-client-manifest.json` 文件包含生成之后的静态资源列表，vue SSR renderer 正是通过这个文件，将当前页面以来的静态资源注入到 `<head>` 标签中。

它是由 [`build/webpack.client.conf.js`](https://github.com/lavas-project/lavas-template-vue-ssr/blob/master/build/webpack.client.conf.js) 文件生成。

一般情况下，这三个文件都不用开发者关注。

### SSR 的状态单例问题

为了降低服务器端的性能开销，我们在 `server.js` 中创建 `renderer` 的时候选择了 `runInNewContext: false` 的方式，因此处理每个请求时，都在同一个上下文，那么就存在 state 单例问题，在这里，我们直接引用 vue 官方的文章来描述这个问题。

原文链接：[避免状态单例](https://ssr.vuejs.org/zh/structure.html)

	当编写纯客户端(client-only)代码时，我们习惯于每次在新的上下文中对代码进行取值。但是，Node.js 服务器是一个长期运行的进程。当我们的代码进入该进程时，它将进行一次取值并留存在内存中。这意味着如果创建一个单例对象，它将在每个传入的请求之间共享。

	如基本示例所示，我们为每个请求创建一个新的根 Vue 实例。这与每个用户在自己的浏览器中使用新应用程序的实例类似。如果我们在多个请求之间使用一个共享的实例，很容易导致交叉请求状态污染(cross-request state pollution)。

	因此，我们不应该直接创建一个应用程序实例，而是应该暴露一个可以重复执行的工厂函数，为每个请求创建新的应用程序实例。

```javascript
// app.js
export function createApp (context) {
    return new Vue({
        data: {
            url: context.url
        },
        template: `<div>访问的 URL 是： {{ url }}</div>`
    });
}
```

```javascript
// app.js
export function createApp() {
    let router = createRouter();
    let store = createStore();
    let app = new Vue({
        router,
        store,
        ...App
    });
    return {app, router, store};
}

```

	同样的规则也适用于 router、store 和 event bus 实例。你不应该直接从模块导出并将其导入到应用程序中，而是需要在 createApp 中创建一个新的实例，并从根 Vue 实例注入。

单例问题很容易被开发者忽视，并且带来的问题还不好排查，值得我们注意。

## 不想用默认的 server.js

SSR 模板提供了在开发环境和生产环境都能使用的 server.js 文件，基于 express，但是这不一定能满足所有开发者的需求，如果我想在生产环境中使用 koa 呢。

既然我们已经知道 vue SSR 的关键部分，那我们编写关键部分代码即可，这个例子用的是 koa next 支持 es2017 的版本，如果 Node.js 版本过低，还得升级，请看 [koa 官网](http://koajs.com/)中的 Installation 章节。

这里是一个使用 Koa 的完整的一个例子，完整可运行，代码的关键之处只有两处

- 创建 `renderer`
- 使用 `renderer.renderToString` 渲染

将这两处掌握了，自定义服务器端渲染就轻而易举了。

```javascript
const fs = require('fs');
const Koa = require('koa');
const Router = require('koa-router');
const send = require('koa-send');
const vueServerRenderer = require('vue-server-renderer');
const bundle = require('./dist/vue-ssr-server-bundle.json');
const clientManifest = require('./dist/vue-ssr-client-manifest.json');

const app = new Koa();
const router = new Router();
const template = fs.readFileSync(__dirname + '/src/index.template.html', 'utf-8');

// 创建 renderer
let renderer = vueServerRenderer.createBundleRenderer(bundle, {clientManifest, template});

// 响应静态文件
router.get('/dist/(.*)', async ctx => await send(ctx, ctx.path));
router.get('/service-worker.js', async ctx => await send(ctx, ctx.path, {root: './dist'}));
router.get('/manifest.json', async ctx => await send(ctx, ctx.path, {root: './static'}));

// 其他的请求都走服务器端渲染
router.all('/', async ctx => {
    ctx.title = 'Lavas';

    ctx.body = await new Promise((resolve, reject) => {
        // 调用 renderer 渲染模板
        renderer.renderToString(ctx, (error, html) => {
            if (error) {
                return reject(error);
            }

            resolve(html);
        });
    });
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000);
```

## 其他的问题

开发者在使用 SSR 的时候难免还会遇到很多其他的问题，这里会讲一些我们自己在开发 SSR 时遇到的问题

### 遇到不支持 SSR 的库怎么办

这个时候有几个办法：

- 找一个支持 SSR 的库，一般情况下，比较困难
- 将用到浏览器环境下才支持的变量或者 API 的地方判断宿主环境，比如 `$vm.isServer` 或者 `process.env.VUE_ENV === 'server'`
- 有时候，一些库初始化就会用到 window，这种情况没办法，要么提 ISSUE 让原作者修改，要么就看一下我们如何通过其他方式解决

下面要讲的这种方法依赖于 webpack，在 Lavas SSR 模板中，已经使用过这种方法。

这里，我们拿 iscroll 作为例子，iscroll 在引进来的时候会进行初始化，导致会出现 window 对象不存在的错误，这里我们用 webpack alias 的方式解决这个问题。

**第一步**，新增一个 [iscroll-ssr.js](https://github.com/lavas-project/lavas-template-vue-ssr/blob/master/src/iscroll-ssr.js)，可以参考 Lavas SSR 中的实现方式，这个文件并不需要什么内容。

```javascript
// iscroll-ssr.js
export default class IScroll {}
```

**第二步**，配置 [webpack.server.conf.js](https://github.com/lavas-project/lavas-template-vue-ssr/blob/master/build/webpack.server.conf.js#L31) 文件，增加 alias，将服务器端渲染使用的 iscroll 映射到新增的 iscroll-ssr.js 文件。

```javascript
// webpack.server.conf.js

alias: {
    'iscroll/build/iscroll-lite$': resolve('./src/iscroll-ssr.js')
}
```
配置了这一步之后，在 vue 和 js 文件中 `import IScroll from 'iscroll/build/iscroll-lite'` 会被替换为 `iscroll-ssr.js` 文件。

但是光这一步还不够，这里有一个深坑，开发者需要避开。

大家都知道，在 Node.js 中通过 `require('iscroll')` 来引入 iscroll，Node.js 会从 `node_modules` 目录中查找，同理在 vue SSR 使用的 `vue-ssr-server-bundle.json` 文件也是一样的从 `node_modules` 中查找 iscroll，那么我们设置的 alias 就不起作用了，默认情况下，我们是将 `node_modules` 目录中的文件通过 [webpack-node-externals](https://github.com/liady/webpack-node-externals) 排除在 `vue-ssr-server-bundle.json` 文件之外的，看下面的 webpack 的配置。我们通过第三步来避开这个深坑。

```javascript
// webpack.server.conf.js
externals: nodeExternals({
    whitelist: [/\.(css|vue)$/]
})
```

**第三步**，把 iscroll 加入 nodeExternals 的白名单，把 iscroll-ssr.js 文件打包进 `vue-ssr-server-bundle.json` 文件，这样 Node.js 就不会再去 node_modules 目录中查找 iscroll 了。

```javascript
// webpack.server.conf.js
externals: nodeExternals({
    whitelist: [/\.(css|vue)$/, /iscroll/]
})
```

还有最后一步。

**第四步**，由于 iscroll-ssr.js 提供的是一个空的 IScroll，那么必然就无法使用，所以在使用的地方要判断当前是否在客户端环境，通过 vue 提供的变量 `$vm.isServer` 或者  `process.env.VUE_ENV` 判断当前运行的环境

```javascript
if (!$vm.isServer) {
    let iscroll = new IScroll();
}

if (process.env.VUE_ENV === 'client') {
    let iscroll = new IScroll();
}
```

通过这四步，就能解决这个问题。
