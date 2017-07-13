# svg-loader 介绍

> [Lavas Basic 模版](https://github.com/lavas-project/lavas-template-vue-basic)并不包含此功能

在[ 如何在项目中使用图标 ](https://lavas.baidu.com/guide/vue/doc/vue/advanced/how-to-use-icon)一节中，我们介绍了开发时引入自定义 svg 图标的方法，例如：

- 将自定义 svg 文件放入指定文件夹下，自动完成注册
- 修改 `config/icon.js` 配置文件引入 svg 格式的 fontawesome 图标

以上修改甚至都不需要重启开发服务器，这一切都是通过模版项目中的 `build/loaders/svg-loader` 完成的，本文将介绍其实现原理。

## loader 是什么

在 webpack 中，[loader](https://doc.webpack-china.org/concepts/loaders/) 用于对模块的源代码进行转换。例如我们熟悉的 [babel-loader](https://github.com/babel/babel-loader) 使用 Babel 转译源代码，[style-loader](https://github.com/webpack-contrib/style-loader) 通过向 html 代码中注入 style 标签添加样式。

在我们的场景中，需要向源代码中注入 svg 的注册代码，此时使用 loader 再合适不过了。

我们使用 webpack 推荐的 [配置方式](https://doc.webpack-china.org/concepts/loaders/#-configuration-)：

- 在 `module.rules` 中添加一条规则，表示我们要修改的是 `src/app.js` 文件
- 在 `resolveLoader.alias` 中声明解析路径，保证 webpack 能找到 loader 路径

```js

// build/webpack.base.conf.js
module: {
    rules: [
        {
            resource: resolve('src/app.js'), // 应用loader的文件
            loader: 'svg-loader',
            enforce: 'pre' // 声明svg-loader最先执行
        }
    ]
},
resolveLoader: {
    alias: {
        'svg-loader': path.join(__dirname, './loaders/svg-loader')
    }
}
```

> info
>
> 在`module.rules`规则中，使用了`enforce: 'pre'`，这是为了保证 svg-loader 的执行时机在所有 loader 之前。例如待修改的`src/app.js`，也满足下面 babel-loader 的规则，将在 svg-loader 处理（注入了使用ES6语法的代码）之后执行。

## 处理流程

现在我们已经完成了 svg-loader 的注册，下面将涉及内部具体的处理流程。

首先要明确我们需要注入的代码内容。之前在[ 如何在项目中使用图标 ](https://lavas.baidu.com/guide/vue/doc/vue/advanced/how-to-use-icon)一节中提到过，我们使用[ vue-awesome](https://github.com/Justineo/vue-awesome)注册自定义 svg 以及使用 svg 格式的 fontawesome 图标。所以以下两类代码就是我们需要注入的：

```js

// 使用 svg 格式的 fontawesome 图标
import 'vue-awesome/icons/envelope';

// 注册自定义 svg 图标
Icon.register({
    myCustomSvg: {
        width: 100,
        height: 100,
        d: 'M...'
    }
});
```

这样 loader 中的逻辑就很清晰了：

- 对于自定义的 svg 文件，遍历 `config/icon.js` 配置文件中设置的 svg 文件夹
- 对于 fontawesome ，遍历 `config/icon.js` 配置文件中的 `icons`

```js
module.exports = function (source) {

    // 从vue-awesome中导入
    if (icons) {
        source += icons.map(name => `import 'vue-awesome/icons/${name}';`).join('');
    }

    // 从svg文件夹中取
    fs.readdirSync(svgDir).forEach(file => {
        let svgName = prefix + path.basename(file, path.extname(file));

        // 注册使用到的svg
        source += `Icon.register(
            {
                '${svgName}': {
                    width: ${parseInt(sizeMatch[1], 10)},
                    height: ${parseInt(sizeMatch[2], 10)},
                    d: '${dMatch[1]}'
                }
            });`;
    });
    return source;
};
```

至此 svg-loader 中的主要流程已经介绍完了，下面我们将关注开发中的文件更新问题。

## 监听文件更新

在开发模式下使用 svg 图标的场景中，会出现两种文件更新情况：
1. 向自定义 svg 文件夹中放入新文件，此时文件夹内容发生更新
2. 添加 fontawesome 图标，此时`config/icon.js`文件内容发生更新

在发生以上两类文件更新时，如果能够自动触发 webpack 重新编译，不需要手动重启服务器，将节省宝贵的开发时间。

webpack 的文件监听机制比较复杂，简单来说就是在 [Compiler](https://github.com/webpack/webpack/blob/master/lib/Compiler.js#L107) 中通过 [Watchpack (底层依赖 chokidar )](https://github.com/webpack/watchpack) 监听了 `compilation.fileDependencies` (单个文件) 和 `compilation.contextDependencies` (文件夹)，两者发生变化均会触发重新编译。

> info
>
> 在开发模式中，webpack-dev-middleware 已经默认 [开启了监听模式](https://doc.webpack-china.org/configuration/watch/)。

在 loader 执行方法中，`this` 指向 [loader 上下文](https://doc.webpack-china.org/api/loaders/)，其中包含了许多重要的属性和方法，这里只关心两个：

- `addDependency()`，向 `compilation.fileDependencies` 中添加监听文件
- `addContextDependency()`，向 `compilation.contextDependencies` 中添加监听文件夹

直接使用这两个方法，就能实现文件更新时触发重新编译了。另外有一点需要注意，`config/icon.js` 中的 `icons` 数组内容发生变动后，需要删除 require 之前的缓存，否则取到的还是旧数据。

```js
const iconConfigPath = require.resolve('../../config/icon');

// 删除require缓存
delete require.cache[iconConfigPath];
const iconConfig = require(iconConfigPath);
const svgDir = iconConfig.svgDir;

// 监听`svg`文件夹变化
this.addContextDependency(svgDir);

// 监听`config/icon.js`文件变化
this.addDependency(iconConfigPath);
```
