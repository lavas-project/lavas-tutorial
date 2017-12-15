# 构建配置

从本篇文档开始，我们将介绍 Lavas 构建、运行中使用的配置项。开发者可以在项目根目录下的 `lavas.config.js` 中定义这些配置项。配置对象的结构大致如下：
```
// lavas.config.js

{
    build: {},
    entry: [],
    middleware: {},
    // 省略其他配置项
}
```

Lavas 内部使用 [Webpack](https://doc.webpack-china.org/concepts/) 进行构建，众所周知 Webpack 功能强大但是配置非常复杂，我们隐藏了大部分构建细节，将构建流程中部分常用特性以配置项的形式暴露给用户，便于快速上手。同时，对于高级开发者，也能通过特殊的配置项将自定义的 Loader 和 Plugin 加入构建流程。

这些在构建过程中使用的 Webpack 相关配置项将放在 `build` 下，下面我们将依次介绍这些配置项。
```
// lavas.config.js

build: {
    path: '',
    publicPath: '',
    // 省略其他配置项
}
```

## path

最终构建产物的输出地址，必须为绝对路径。如下配置将输出构建产物到 `dist` 文件夹下：
```
path: path.resolve(__dirname, 'dist')
```
等同于 Webpack 配置中的 [output.path](https://doc.webpack-china.org/configuration/output/#output-path)。

## publicPath

在静态资源路径之前添加的前缀。默认值为 `'/'`。
```
publicPath: '/'
```

例如在使用 CDN 场景下，可以使用如下配置：
```
publicPath: '//cdn.example.com/assets/'
```

更多使用例子可参考 Webpack 配置中的 [output.publicPath](https://doc.webpack-china.org/configuration/output/#output-publicpath)。

## filenames

Webpack 可以指定输出静态资源（JS CSS FONT IMG）的文件名，其中可以使用例如 `[hash]` 这样的模板字符串。
Lavas 中使用的默认值如下：
```
filenames: {
    entry: 'js/[name].[chunkhash:8].js',
    vue: 'js/vue.[chunkhash:8].js',
    vendor: 'js/vendor.[chunkhash:8].js',
    chunk: 'js/[name].[chunkhash:8].js',
    css: 'css/[name].[contenthash:8].css',
    img: 'img/[name].[hash:8].[ext]',
    fonts: 'fonts/[name].[hash:8].[ext]'
}
```

其中：

* **entry** entry chunk。将影响各个入口文件名。 可参考 Webpack 中的 [output.filename](https://doc.webpack-china.org/configuration/output/#output-filename)。
* **vue** 我们将 Vue 相关的依赖合并成一个 chunk。包括 vue、vue-router、vuex 和 vue-meta。
* **vendor** 包含其他第三方依赖。
* **chunk** async chunk。将影响非入口文件名。可参考 Webpack 中的 [output.chunkFilename](https://doc.webpack-china.org/configuration/output/#output-chunkfilename)。
* **css** 样式文件。由于使用了 [ExtractTextWebpackPlugin](https://doc.webpack-china.org/plugins/extract-text-webpack-plugin)从 JS 中提取样式，必须使用 `[contenthash]` 而非 `[hash]` 或者 `[chunkhash]`。
* **img** 图片。
* **fonts** 字体文件。

更多模板字符串示例及其使用场景可以参考 Webpack [output.filename](https://doc.webpack-china.org/configuration/output/#output-filename)。

## cssExtract

在使用 [ExtractTextWebpackPlugin](https://doc.webpack-china.org/plugins/extract-text-webpack-plugin)从 JS 中提取样式时，需要设置 Loader 和 Plugin。由于分离样式会造成额外的编译开销，Lavas 默认在开发模式中关闭这一特性，在生产环境打开。
```
cssExtract: true
```

## cssMinimize & cssSourceMap

是否需要对 CSS 文件进行压缩以及生成 source-map。
```
cssMinimize: true,
cssSourceMap: true
```

这两个参数最终将传递给 [css-loader](https://github.com/webpack-contrib/css-loader)。
```
loader: 'css-loader',
options: {
    minimize: true,
    sourceMap: true
}
```

## jsSourceMap

是否需要对 JS 文件生成 source-map，便于开发模式调试以及生产环境排查错误，默认开启。
```
jsSourceMap: true
```

Webpack 支持通过 `devtool` 配置项生成多种 [source-map](https://doc.webpack-china.org/configuration/devtool/)。这些不同格式的 source-map 在生成速度，是否内联在源文件中等等方面都有显著差异，需要使用者根据具体场景选择合适的格式。
在开发模式中，由于代码经常发生变动，我们通常会选择生成速度快，可以接受内联从而增加源文件体积的代价。而在生产环境中，我们通常选择生成独立的 source-map 文件，此时生成速度就可以忽略了。

在 Lavas 中开启这个配置后将作用于以下两种场景：

1. 开发模式中选择 `cheap-module-eval-source-map`。这种格式只显示行号不显示列号，重新生成速度很快。
2. 生产环境中选择 `nosources-source-map`。这种格式下生成的独立 source-map 不会暴露源文件内容，只会显示错误堆栈信息。同时通过 [UglifyJsPlugin](https://doc.webpack-china.org/plugins/uglifyjs-webpack-plugin) 在压缩 JS 文件的同时生成 sourceMap，内部通过设置插件的 [sourcemap 选项](https://doc.webpack-china.org/plugins/uglifyjs-webpack-plugin/#sourcemap)实现。

## bundleAnalyzerReport

[Webpack Bundle Analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer) 提供了可视化图表这样的直观方式，帮助开发者分析构建产物中可能出现的问题，例如重复引入、不必要的依赖。
```
// 默认配置，启动 localhost:8888 服务器展示网页
bundleAnalyzerReport: true

// 自定义配置
bundleAnalyzerReport: {
    analyzerMode: 'server',
    analyzerHost: '127.0.0.1',
    analyzerPort: 8888,
    // 省略其他配置
}
```

Lavas 默认关闭这一配置。开启后，运行 `lavas build`，将自动启动服务器并打开网页，以下是 Lavas 模板项目的分析结果：
![bundle-analyzer 分析结果](../../images/bundle-analyzer.png)

## defines

在构建时我们常常需要使用全局常量，在 Webpack 中可以通过 [DefinePlugin](https://doc.webpack-china.org/plugins/define-plugin/) 插件定义这些常量。Lavas 提供三组命名空间，分别是 SSR 中服务端使用的 `server`，客户端使用的 `client` 以及两者共用的 `base`。另外需要注意的是常量的值必须包含字符串本身内的实际引号，可以使用单引号包含双引号或者 `JSON.stringify()`。
```
defines: {
    base: {
        'MY_CONSTANT': '"VALUE"'
    },
    client: {},
    server: {}
}
```

一个常见的使用场景是，需要根据开发环境和生产环境定义不同的 URL。由于在启动 Lavas 时已经设置了环境变量 `process.env.NODE_ENV`，在 `lavas.config.js` 中可以这样做：
```
// lavas.config.js
const isProd = process.env.NODE_ENV === 'production';

module.exports = {
    build: {
        defines: {
            base: {
                PASSPORT_URL: isProd
                    ? JSON.stringify('https://wappass.example.com/passport')
                    : JSON.stringify('https://wappass.qatest.example.com/passport')
            }
        },
```

另外，Lavas 已经内置了以下两组全局常量 `process.env.VUE_ENV` 和 `process.env.NODE_ENV`，可以直接在项目中使用，不需要开发者重复定义：
```
// 在同构应用当前处于 client 或者 server 端
'process.env.VUE_ENV': '"client"',

// 当前应用处于开发模式 development 或者生产模式 production
'process.env.NODE_ENV': '"development"'
```

## alias

在 Webpack 中解析模块时，我们常常使用 alias 定义路径的[简写别名](https://doc.webpack-china.org/configuration/resolve/#resolve-alias)，便于更加简便地引用模块。

Lavas 提供了 `alias` 下三组命名空间，分别是 SSR 中服务端使用的 `server`，客户端使用的 `client` 以及两者共用的 `base`。
```
alias: {
    base: {},
    client: {},
    server: {}
}
```

另外，Lavas 已经内置了两组别名，开发者不需要重复定义。例如如果想引用项目根目录下 `components` 文件夹中的组件，只需要使用 `import MyComponent from '@/components/MyComponent'`：
```
    '@': 指向项目根目录,
    '$': 指向 .lavas 目录
```

## plugins

在使用 Webpack 构建时，各种插件是必不可少的，对于开发者自定义的插件，Lavas 提供了 `plugins` 下三组命名空间，分别是 SSR 中服务端使用的 `server`，客户端使用的 `client` 以及两者共用的 `base`。有一点需要注意，自定义插件将添加到 Lavas 已有插件列表之后，如果对于插件添加顺序有要求，可以参考 `extend` 配置项，进行更精确的添加。
```
plugins: {
    base: [],
    client: [],
    server: []
}
```

## extend

为了给予开发者更大的灵活度，能自由修改 Webpack 配置对象，Lavas 提供了 `extend` 方法。
该方法参数说明如下：
* `config` Webpack 配置对象。
* `options.type` Webpack 配置对象类型，一共有三种：`client` 供客户端使用，`server` 供服务端使用，而 `base` 为两者所共用。
* `options.env` 当前构建环境变量，取值有两种：`development|production`。

例如我们想增加 `vue-style-variables-loader` 来处理 `.vue` 文件，可以这么做：
```
extend(config, {type, env}) {
    if (type === 'base') {
        let vueRule = config.module.rules[0];
        vueRule.use.push({
            loader: 'vue-style-variables-loader',
            options: {
                variablesFiles: [
                    path.join(__dirname, 'assets/styles/variables.styl')
                ]
            }
        });
    }
}
```

## compress

注意：该配置项只有 SSR 模式下生效，MPA 模式下可以忽略。

在 SSR 模式下是否启用 gzip，通过内置的 [compress 中间件](https://github.com/expressjs/compression)实现。Lavas 默认在开发模式中关闭这一特性，在生产环境打开。
```
compress: false
```

## nodeExternalsWhitelist

注意：该配置项只有 SSR 模式下生效，MPA 模式下可以忽略。

在 SSR 模式下，通常我们不希望将 `node_modules` 中的依赖打包进 server bundle 中，因此需要使用 Webpack [externals](https://doc.webpack-china.org/configuration/externals/) 配置项。Lavas 已经通过 [Webpack node modules externals](https://github.com/liady/webpack-node-externals) 将 `node_modules` 全部排除。但是在某些场景下，我们还是需要将部分特定的依赖打包进来，这时就需要使用[白名单](https://github.com/liady/webpack-node-externals#optionswhitelist-)了：
```
nodeExternalsWhitelist: []
```

例如在服务端渲染场景下常常遇到的一个问题是，某些第三方依赖使用了 `document`, `window` 这样在 Node.js 环境中不存在的对象。为了保证服务端渲染正常运行，通常使用 `resolve.alias` 引导 Webpack 使用空的 stub 对象，此时一定要同时在 `nodeExternalsWhitelist` 中加入该依赖。

## ssrCopy

注意：该配置项只有 SSR 模式下生效，MPA 模式下可以忽略。

在 SSR 模式下，Lavas 除了将构建产物输出到例如 `dist` 文件夹中，还可以将例如 `node_modules`，线上脚本等文件拷贝到里面。这样 `dist` 文件夹可以作为一个可单独运行的包，移动到任意位置：
```
ssrCopy: isDev ? [] : [
    {
        src: 'server.prod.js'
    },
    {
        src: 'node_modules'
    },
    {
        src: 'package.json'
    }
]
```

虽然从功能上看都是拷贝文件，但是这些文件并不会经过 CopyWebpackPlugin 处理，这一点不同于 `/static` 文件夹。

## watch

Lavas 在开发模式下使用了 webpack-dev-middleware。得益于自带的热加载功能，很多源文件的修改会自动触发 Webpack 重新编译，不需要开发者重启开发服务器。Lavas 扩充了这个功能，修改以下文件，也会触发重新编译：

* `/pages` 下增加删除修改路由组件。
* `lavas.config.js` 配置内容发生修改。
* SSR 模式下模板内容发生修改。

整个重新编译过程中不需要开发者关闭重启服务，也就是说从 MPA 模式切换到 SSR 模式也只需要修改配置后等待编译完成。另外，如果想监控自定义文件、文件夹，在它们发生修改时也触发重新编译，可以通过 `watch` 配置项传入文件列表：
```
watch: [
    '/foo/bar' // 自定义文件
]
```

## development & production

对于以上配置项，如果需要在开发模式和生产环境启用不同的值，可以使用两个特殊的配置项 `development` 和 `production`。
例如想在开发模式关闭 `cssExtract` 分离样式而在生产环境开启，可以这么做：
```
build: {
    // 省略其他配置项
},
development: {
    build: {
        cssExtract: false
    }
},
production: {
    build: {
        cssExtract: true
    }
}
```
