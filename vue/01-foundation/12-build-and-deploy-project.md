# 构建部署工程

和其他基于 vue 模版项目一样，构建和部署都十分简单。

## 生产环境构建

生产环境构建出的所有静态资源默认输出在 /dist 文件夹下。

```bash
npm run build
```

执行命令后，在控制台的输出主要分成两部分，第一部分是 sw-precache 缓存的静态资源列表。
![sw-precache缓存的静态资源列表](./images/build-output-sw.png)

第二部分就是 vue 模版项目通用的最终生成的静态资源列表。
![构建的静态资源列表](./images/build-output-assets.png)

## 部署到服务器

由于模版项目默认使用了 vue-router 的 HTML5 History 模式，需要将所有 URL 指向 index.html，避免在单页面项目中出现 404 情况，这就需要在服务器端进行一些配置。

具体配置可参考[ vue-router 官方文档中 Apache，nginx 和 Express 的配置](https://router.vuejs.org/en/essentials/history-mode.html)。
