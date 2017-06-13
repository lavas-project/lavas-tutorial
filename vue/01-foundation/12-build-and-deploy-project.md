# 构建部署工程

和其他基于 vue 模版项目一样，构建和部署都十分简单。

## 构建

```bash
npm run build
```

默认输出的文件存放在`/dist`文件夹下。

## 部署

由于模版项目默认使用了 HTML5 History 模式，所以需要在服务器端进行一些配置。

具体配置可参考[vue-router官方文档中Apache，nginx和Express的配置](https://router.vuejs.org/en/essentials/history-mode.html)。
