# vue-skeleton-webpack-plugin 介绍

> info
>
> 这是一个基于 vue 的 webpack 插件，为单页和多页应用生成 skeleton，提升首屏展示体验。

如果您还不了解 skeleton，可以参考[App Skeleton 介绍](https://lavas.baidu.com/guide/vue/doc/vue/advanced/skeleton)一文。

github 地址：[https://github.com/lavas-project/vue-skeleton-webpack-plugin](https://github.com/lavas-project/vue-skeleton-webpack-plugin)

## 问题背景

参考[饿了么的 PWA 升级实践](https://huangxuan.me/2017/07/12/upgrading-eleme-to-pwa/#在构建时使用-vue-预渲染骨架屏)一文，我们希望在构建时渲染 skeleton 组件，将渲染 DOM 插入 html 的挂载点中，同时将使用的样式通过 style 标签内联。这样在前端 JS 渲染完成之前，用户将看到页面的大致骨架，感知到页面是正在加载的。

我们当然可以选择在开发时直接将页面骨架内容写入 html 模板中，但是这会带来两个问题：
1. 开发 skeleton 与其他组件体验不一致。
2. 多页应用中多个页面可能共用同一个 html 模板，而又有独立的 skeleton。

下面我们将看看插件在具体实现中是如何解决这两个问题的。

## 实现思路

我们希望能够保证一致的开发体验，开发 skeleton 和其他组件没有任何不同。而且开发者不需要关心渲染结果是如何被注入 html 的。
下面我们将从渲染和注入这两方面展开介绍。

### 渲染组件

我们使用 vue 的[服务端渲染](https://ssr.vuejs.org/zh/)功能，接受 webpack 配置对象作为输入，输出渲染的 DOM 和样式。

一个典型的用于服务端渲染的 webpack 配置对象如下，其中 entry 入口文件中使用了 skeleton 组件：
```js
{
    target: 'node',
    devtool: false,
    entry: resolve('./src/entry-skeleton.js'), // 多页应用中传入数组
    output: Object.assign({}, baseWebpackConfig.output, {
        libraryTarget: 'commonjs2'
    }),
    externals: nodeExternals({
        whitelist: /\.css$/
    }),
    plugins: []
}
```

> info
>
> 多页中的 webpack 配置对象示例，可参考[多页测试用例](https://github.com/lavas-project/vue-skeleton-webpack-plugin/tree/master/examples/multipage)或者[Lavas MPA 模板](https://github.com/lavas-project/lavas-template-vue-mpa)。

webpack 将使用传入的配置对象进行编译，由于我们不需要将最终产物保存在硬盘中，使用内存文件系统[memory-fs](https://github.com/webpack/memory-fs)能够减少不必要的I/O开销。最终会生成一个 bundle 文件，使用`createBundleRenderer`创建一个 renderer，就可以在 Node.js 环境得到渲染结果了。

```js
const createBundleRenderer = require('vue-server-renderer').createBundleRenderer;
let bundle = mfs.readFileSync(outputPath, 'utf-8');
// 创建 renderer
let renderer = createBundleRenderer(bundle);
// 渲染得到 html
renderer.renderToString({}, (err, skeletonHtml) => {
    if (err) {
        reject(err);
    }
    else {
        resolve({skeletonHtml, skeletonCss});
    }
});
```

另外，为了将样式从 JS 文件中分离，我们使用了[ ExtractTextPlugin](https://github.com/webpack-contrib/extract-text-webpack-plugin)插件，将样式内容输出到单独的文件中。
```js
// vue-skeleton-webpack-plugin/src/ssr.js

// 加入 ExtractTextPlugin 插件
serverWebpackConfig.plugins.push(new ExtractTextPlugin({
    filename: outputCssBasename
}));
```

至此，我们已经得到了全部渲染结果，剩下的就是注入时机了。

### 注入渲染结果

关于渲染结果的注入时机，我们参考[html-webpack-plugin的事件说明](https://github.com/jantimon/html-webpack-plugin#events)，选择在`html-webpack-plugin-before-html-processing`事件回调函数中进行。

渲染结果中包含 DOM 结构和样式两部分，样式可以直接插入`</head>`之前，而 DOM 的插入与挂载点相关，默认使用`<div id="app">`，当然插件使用者可以通过参数传入。

在多页应用中，相比单页情况会变的稍稍复杂。多页项目中通常会引入多个 html-webpack-plugin，例如我们在[Lavas MPA 模板](https://github.com/lavas-project/lavas-template-vue-mpa)中使用的[ multipage插件](https://github.com/mutualofomaha/multipage-webpack-plugin)就是如此，这就会导致`html-webpack-plugin-before-html-processing`事件被多次触发。我们需要在每次事件触发时识别出当前处理的入口文件，执行 webpack 编译当前页面对应的入口文件，渲染对应的 skeleton 组件。

查找当前处理的入口文件过程如下：
```js
// vue-skeleton-webpack-plugin/src/index.js

// 当前页面使用的所有 chunks
let usedChunks = htmlPluginData.plugin.options.chunks;
let entryKey;
// chunks 和所有入口文件的交集就是当前待处理的入口文件
if (Array.isArray(usedChunks)) {
    entryKey = Object.keys(skeletonEntries);
    entryKey = entryKey.filter(v => usedChunks.indexOf(v) > -1)[0];
}
// 设置当前的 webpack 配置对象的入口文件和结果输出文件
webpackConfig.entry = skeletonEntries[entryKey];
webpackConfig.output.filename = `skeleton-${entryKey}.js`;
// 使用配置对象进行服务端渲染
ssr(webpackConfig).then(({skeletonHtml, skeletonCss}) => {});
```

### 开发模式下插入路由

由于 skeleton 的渲染结果在 JS 前端渲染完成后就会被替换，如何在开发时方便的查看呢？
使用浏览器开发工具设置断点，阻塞前端渲染可以做到，但如果能在开发模式中插入 skeleton 对应的路由规则，使多个页面的 skeleton 能像其他路由组件一样被访问，将使开发调试变得更加方便。

向路由文件中注入代码的工作将在[ loader](https://github.com/lavas-project/vue-skeleton-webpack-plugin/blob/master/src/loader.js)中完成。

首先明确注入内容，我们希望通过路由组件的形式访问 skeleton，那么首先需要引入各个 skeleton 组件，然后增加对应的路由规则。具体到注入的代码，类似这样：
```js
// router.js

// 引入 skeleton 组件
import Skeleton from '@/pages/Skeleton.vue'

// 插入routes
routes: [
    {
        path: '/skeleton',
        name: 'skeleton',
        component: Skeleton
    }
    // ...其余路由规则
]
```

在多页应用中，使用者可以通过占位符设置依赖语句和路由规则的模板，loader 在运行时会使用这些模板，用真实的 skeleton 名称替换掉占位符，[插入多条语句](https://github.com/lavas-project/vue-skeleton-webpack-plugin/blob/master/src/loader.js#L27-L39)。

> info
>
> 多页中的具体应用示例，可参考[多页测试用例](https://github.com/lavas-project/vue-skeleton-webpack-plugin/tree/master/examples/multipage)或者[Lavas MPA 模板](https://github.com/lavas-project/lavas-template-vue-mpa)。

## 参数说明

插件和 loader 使用的参数如下：

### SkeletonWebpackPlugin

- webpackConfig *必填*，渲染 skeleton 的 webpack 配置对象
- insertAfter *选填*，渲染 DOM 结果插入位置，默认值为`'<div id="app">'`

### SkeletonWebpackPlugin.loader

参数分为两类：
1. [ webpack模块规则](https://doc.webpack-china.org/configuration/module/#rule)，skeleton 对应的路由将被插入路由文件中，所以需要指定一个或多个路由文件，使用`resource/include/test`皆可指定 loader 应用的文件。
2. `options` 将被传入 loader 中的参数对象，包含以下属性：
    - entry *必填*，支持字符串和数组类型，对应页面入口的名称
    - importTemplate *选填*，引入 skeleton 组件的表达式，默认值为`'import [nameCap] from \'@/pages/[nameCap].vue\';'`
    - routePathTemplate *选填*，路由路径，默认值为`'/skeleton-[name]'`
    - insertAfter *选填*，路由插入位置，默认值为`'routes: ['`

在`importTemplate`和`routePathTemplate`中可以使用以下占位符：
- `[name]` 和`entry`保持一致
- `[nameCap]` `entry`首字母大写

例如使用以下配置，将向路由文件`router.js`中插入`'import Page1 from \'@/pages/Page1.vue\';'`和`'import Page2 from \'@/pages/Page2.vue\';'`两条语句。
同时生成`/skeleton-page1`和`/skeleton-page2`两条路由规则。
```js
{
    resource: 'router.js',
    options: {
        entry: ['page1', 'page2'],
        importTemplate: 'import [nameCap] from \'@/pages/[nameCap].vue\';',
        routePathTemplate: '/skeleton-[name]'
    }
}
```

更多详细说明可参考[ github上插件的参数说明部分](https://github.com/lavas-project/vue-skeleton-webpack-plugin#参数说明)。

## 贡献代码

在开发中遇到任何问题，都欢迎提出[ ISSUE](https://github.com/lavas-project/vue-skeleton-webpack-plugin/issues)讨论。

您也可以帮助我们完善[测试用例](https://github.com/lavas-project/vue-skeleton-webpack-plugin/tree/master/examples)。
