# router-loader 介绍

> 仅有[Lavas MPA 模板](https://github.com/lavas-project/lavas-template-vue-mpa)包含此功能

## 问题背景

在单页应用中我们使用 vue-router 进行前端路由跳转，而多页应用可以看成多个单页应用，每个单页都可以有各自独立的路由，那么如何做到在各个单页之间进行跳转呢？

例如想从A页面跳转到B页面，发现目标路由规则并不在A页面的规则集中，此时肯定不能展示404页面，需要识别出这是一个有效的路由规则，通过`window.location.href`而非 vue-router 进行强制跳转。

这就要求我们在构建时收集所有页面使用的路由规则，形成一个项目中有效的路由规则全集。各个页面遇到不匹配的路由时，都需要去这个全集中查看，匹配了其中的某条规则才进行强制跳转，否则依然展示404页面。

## 具体实现

依据以上实现思路，首先在通用路由生成函数中，加入通用路由处理规则`*`，在`beforeEnter`钩子中使用`validateRoute`校验目标路由是否匹配项目中使用到的合理规则，校验失败依旧展示404页面。
```js
// src/router.js

export function createRouter({routes = []}) {
    const router = new Router({
        routes: [
            ...routes,
            {
                path: '*',
                component: NotFound,
                beforeEnter(to, from, next) {
                    // 有效路由，跳转
                    if (validateRoute(to.fullPath)) {
                        window.location.href = to.fullPath;
                        return;
                    }
                    next(); // 无效路由，展示404页面
                }
```

在`validateRoute`方法中，由于动态路由的存在，需要对动态参数进行替换，使用正则进行匹配。
```js
function validateRoute(path) {
    return allRoutes.includes(path)
        || allRoutes.some(route => {
            // 生成路由路径对应的正则表达式 /detail/:id => /^\/detail\/[^\/]+\/?$/
            let routeRegex = new RegExp(`^${route.replace(/\/:[^\/]*/g, '/[^\/]+')}\/?$`);
            return routeRegex.test(path);
        });
}
```

对于路由规则全集`allRoutes`，将由 router-loader 负责收集。

### router-loader 实现

首先明确路由全集的注入点，在`src/router.js`中，我们需要将路由全集放入数组`allRoutes`中。
```js
// src/router.js

const allRoutes = [];
```

在[Lavas MPA 模板](https://github.com/lavas-project/lavas-template-vue-mpa)中，每个页面有独立的路由文件：
``` bash
lavas-template-vue-mpa
    |---src
        |---pages 页面存放目录
            |---detail 详情页模块
                |--- Detail.skeleton.vue 构建时渲染的skeleton组件
                |--- Detail.vue 路由组件
                |--- entry-skeleton.js skeleton入口
                |--- entry.js entry入口
                |--- index.html 页面模板，供htmlWebpackPlugin使用
                |--- router.js 单页面使用的路由
            |---home 主页模块
            |---search 搜索页模块
        |---...省略其他目录
```

router-loader 只需要遍历各个页面的路由文件，从内容中提取出使用的路由规则路径（例如`path: '/home'`），拼接后注入。
另外，在[ svg-loader介绍](https://lavas.baidu.com/guide/vue/doc/vue/webpack/svg-loader)一文中提到了监听文件更新的问题，这里我们也将每个路由文件加入了 webpack 监听文件列表，方便开发使用。

```js
// src/router.js插入点
let pos = source.indexOf(INSERT_POSITION) + INSERT_POSITION.length;
// 各个页面路由路径
let entryRouters = utils.getEntries('./src/pages', 'router.js');
let routePaths = [];
for (let entryName in entryRouters) {
    if (entryRouters.hasOwnProperty(entryName)) {
        let routerPath = path.resolve(__dirname, '../../', entryRouters[entryName]);
        // 读取各个页面路由文件内容
        let content = fs.readFileSync(routerPath, 'utf8');
        // 解析内容中的路径
        routePaths = routePaths.concat(extractRoutePaths(content));
        // 加入文件监听列表
        this.addDependency(routerPath);
    }
}
// 拼接所有路由路径，注入
return insertAt(source, routePaths.join(','), pos);
```

