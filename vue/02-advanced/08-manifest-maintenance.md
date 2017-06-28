# 维护 manifest.json

开始之前，您可以查看[添加到屏幕](https://pwa.baidu.com/doc/engage-retain-users/add-to-home-screen/01-introduction)相关内容，快速掌握相关基础。

为了实现 PWA 应用添加至桌面的功能，我们需要准备 `manifest.json` 文件配置应用的图标、名称等信息。 **项目中这些信息在哪里配置呢?**


## 如何配置 ？

项目的配置文件一般在 config 文件夹下，这里的配置也不例外，就在 `config/manifest.js` 中，开发中我们一般只需要关注该文件的配置内容即可满足开发需求。 config/manifest.js 中配置了应用添加到屏幕所需的相关字段（这里给出部分必须的参数，具体的可以参看[配置文档及示例](https://pwa.baidu.com/doc/engage-retain-users/add-to-home-screen/01-introduction)），根据这些配置，项目 build 后会自动生成所需的 manifest.json 文件。下面是具体的配置代码及简单的介绍：

``` js
module.exports = {
    // 生成的文件名称，如果要修改，引用路径要一起修改
    fileName: 'manifest.json',

    // 完整名称
    name: 'pwa-news',

    // 短名称
    shortName: 'pwa-news',

    // 应用图标列表
    icons: [
        {
            src: '/static/img/icons/android-chrome-192x192.png',
            sizes: '192x192',
            type: 'image/png'
        },
        {
            src: '/static/img/icons/android-chrome-512x512.png',
            sizes: '512x512',
            type: 'image/png'
        }
    ],

    // 应用启动地址
    startUrl: '/',

    // 指定 PWA 从主屏幕点击启动后的显示类型
    display: 'standalone',

    // 启动画面的背景颜色
    backgroundColor: '#000000',

    // 指定PWA的主题颜色
    themeColor: '#3e98f0'
};
```

## 如何自动生成 ？

在 webpack 打包时，我们提供了一个 `build/plugins/manifest-webpack-plugin` 的插件来完成这个任务，生成规范的 `manifest.json` 文件。该插件干了几件事：

* 将配置文件中的键名驼峰式转换成以 `_` 连接的形式，并替换原对象，得到符合规范的配置内容
* 将整理好的配置内容 写入指定路径下的 `manifest.json` 中待引用
* 替换 index.html 中的主题背景色 theme-color 为配置项中的主题色



## 如何测试验证 ？

* 本地起服务，通过安卓手机设置手动代理，在 Chrome 浏览器中通过 `localhost` 访问服务站点，点击功能菜单中的 “添加到主屏幕”，在桌面检验是否添加成功，配置信息是否生效

* 项目 build 后部署启动过后（一定要 https 哦），通过安卓手机在 Chrome 浏览器中访问站点，点击功能菜单中的 “添加到主屏幕”，在桌面检验是否添加成功，配置信息是否生效


## 小结

了解上面几步，有没有感觉了然于胸，使用就是那么方便，分分钟就搞定啦！
