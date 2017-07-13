# 构建部署工程

和其他基于 vue 模版项目一样，构建和部署都十分简单。

## 生产环境构建

生产环境构建出的所有静态资源默认输出在 `/dist` 文件夹下。

```npm
$ npm run build
```

执行命令后，在控制台的输出主要分成两部分，第一部分是 sw-precache 缓存的静态资源列表。
![sw-precache 缓存的静态资源列表](./images/build-output-sw.png)

第二部分就是 vue 模版项目通用的最终生成的静态资源列表。
![构建的静态资源列表](./images/build-output-assets.png)

## 部署到服务器

由于 Lavas 导出模版中项目默认使用了 vue-router 的 `HTML5 History` 模式（而非采用 hash 模式），我们还需要做一些服务器端的配置。

（1）为了避免在单页面项目中出现 404 情况，就要保证将所有 URL 指向 index.html，服务器端需要进行这一配置（目前的 basic、AppShell 模板均为单页项目）。以 nginx 为例，安装好 nginx 后，可以在 nginx.conf 文件中增加一个 server, 配置好端口，启动项目路径 root (即 build 后的 dist 文件夹地址)，配置下面的 location，启动 nginx 即可。

```js
listen       8848;
server_name  localhost;
root /users/eral/workspace/pwa-shell/dist/;

location / {
    try_files $uri $uri/ /index.html;
}
```

> 注意
>
> 因为这么做以后，你的服务器就不再返回 404 错误页面，因为对于所有路径都会返回 index.html 文件。为了避免这种情况，你应该在 Vue 应用里面覆盖所有的路由情况，然后在给出一个 404 页面。

```js
const router = new VueRouter({
    mode: 'history',
    routes: [
        {
            path: '*',
            component: NotFoundComponent
        }
    ]
})
```

(2) 同理，多页项目模板中，我们需要在后端服务器配置多个路由指定到不同的页面(目前的 mpa 模板为多页)，才能正常的访问。同样以 nginx 为例，可以在 nginx.conf 文件中配置多条 location，其他同上，启动 nginx 即可。

```js
listen       8848;
server_name  localhost;
root /users/eral/workspace/pwa-shell/dist/;

location /home {
    try_files $uri $uri/ /home.html;
}

location /detail {
    try_files $uri $uri/ /detail.html;
}

location /search {
    try_files $uri $uri/ /search.html;
}
```



具体配置可参考 [vue-router 官方文档中 Apache，nginx 和 Express 的配置](https://router.vuejs.org/zh-cn/essentials/history-mode.html)。

