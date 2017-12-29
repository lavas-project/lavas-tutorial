# 多个 Lavas 项目整合

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

## 实现方式

为了表述方便，我们来创建一个简单但可能很实用的需求，通过解决这个需求来学习如何整合。

### 示例需求

我们假设开发者存在这样的开发需求：一个电商站点从业务模块角度能分成两部分，分别是：

* `/user/*` 部分，用户信息浏览/注册/修改等等相关，采用 SPA 模式渲染。一般这类用户信息敏感的内容不需要考虑 SEO，也就不需要 SSR。

* 剩余部分，是站点的主要内容，例如 `/` 展示站点首页，`/detail/*` 查看商品详情，`/search/*` 进行商品搜索等等。这部分考虑到 SEO 需求，使用 SSR 模式进行渲染。

我们假设开发者已经分别为两者开发了单独的 Lavas 应用。前者名为 lavas-user，后者名为 lavas-main。我们需要这么几个步骤：

1. 修改基础路由和端口号互相区分

2. 提取共享模块避免重复 (可选)

3. 配置流量分发服务器

### 修改基础路由和端口号

打开 `/lavas.config.js` 配置文件的 `router` 段，可以找到 `base` 配置项

```javascript
// ...
router: {
    mode: 'history',
    base: '/',
    pageTransition: {
        enable: false
    }
}
// ...
```

观察示例需求的两个模块，lavas-user 拥有明显的 URL 特征 ( /user 开头)，而 lavas-main 没有。因此我们修改 lavas-user 的 `base` 为 `/user/` (__不要遗漏最后的 `/` ！__)，lavas-main 不作修改。

`base` 配置项是 vue-router 的一个配置项，用以设定基准路由。修改后的 lavas-main，原本使用 `/view` 的路由就变成了 `/user/view`， 原本使用 `/register` 就变成了 `/user/register`，以此类推。

因为路由发生了变化，如果项目中有使用相对路径引用静态资源，也需要一并修改。例如如果在 `/core/index.html.tmpl` 中引用了 `/static/some-script.js`，那么这里就要改成 `<%=baseUrl%>static/some-script.js`。这部分可以参考[index.html.tmpl 相关文档](/guide/v2/advanced/core#index.html.tmpl)

为了同时启动两个 Lavas 项目，必须保证他们启动在不同的端口号。因此我们还需要修改 `/server.prod.js`，如下：

```javascript
const LavasCore = require('lavas');
const express = require('express');
const app = express();

let port = 8001;
// 省略其他内容
```

通过这个配置我们使两个项目启动在不同端口。例如将 lavas-user 启动在 8001，lavas-main 启动在 8002。

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

### 配置流量分发服务器

最后一步是在两个 Lavas 服务之前搭建一个分发服务器，对不同的 URL 转发到不同的 Lavas 服务。以 nginx 为例，我们使用如下配置：

```
server {
    listen          80
    server_name     localhost

    location /user/ {
        proxy_pass                  http://localhost:8001;
        proxy_set_header Host       $host;
        proxy_set_header X-Real-IP  $remote_addr;
    }

    location / {
        proxy_pass                  http://localhost:8002;
        proxy_set_header Host       $host;
        proxy_set_header X-Real-IP  $remote_addr;
    }
}
```

就可以成功将流量转发到两个 Lavas 服务上了。
