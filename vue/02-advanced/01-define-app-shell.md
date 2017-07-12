# App Shell 调整及扩展

## App Shell 模型

App Shell 架构是构建 PWA 应用的一种方式，它通常提供了一个最基本的 WebApp 框架，包括应用的头部、底部、菜单栏等结构。顾名思义，我们可以把它理解成应用的一个「空壳」，这个「空壳」仅包含页面框架所需的最基本的 HTML 片段，CSS 和 JavaScript，这样一来，用户重复打开应用时就能迅速地看到 WebApp 的基本界面，只需要从网络中请求、加载必要的内容。我们使用 service worker 对 App Shell 做离线缓存，以便它可以在离线时正常展现，达到类似 Native App 的体验。

譬如项目的简单示例：新闻 Demo，就演示了这一架构，我们将头部及导航栏作为 Shell，其余部分为动态更新的内容，如下图。

![App Shell 示例](./images/app-shell-1.png)

开发一个 App Shell, 我们通常要注意以下几点：

- 将动态内容与 Shell 分离
- 尽可能使用较少的数据，实现快速加载
- 使用本地缓存中的静态资源

明确以上内容之后，我们就可以着手开发、定制自己的 App Shell 了。


## 调整及扩展 App Shell

你可以从零开始开发自己的 App Shell，但为了降低开发成本，Lavas 已经为你准备好了一个比较通用的 App Shell，你可以直接使用它，也可以在它的基础上进行调整及扩展。

在 [Lavas 的命令行工具](https://github.com/lavas-project/lavas) 提供的多种类型模板中，我们选择 `App Shell` 模板，
它集成的 App Shell 包含了典型的页面头部、页面底部导航、侧边展开栏等基本结构，如下图所示。

![appshell1](./images/app-shell-3.png)

![appshell2](./images/app-shell-4.png)

整个 App Shell 结构由不同的组件组成，在 `src/components` 目录中进行管理。

```
src/components
    ├── AppBottomNavigator.vue
    ├── AppHeader.vue
    ├── AppMask.vue
    ├── AppSidebar.vue
    ├── ProgressBar.vue
    └── Sidebar.vue
```

为了方便 App Shell 与页面之间的交互，我们将 App Shell 各组件的状态放在 vuex 的 store 中统一管理，具体实现在 `src/store/modules/app-shell.js`， 它将 Shell 各组件划分成不同的 modules，包含了组件的 state, actions, mutations 等信息。这样一来，页面组件可以通过 `mapStates/mapActions` 访问当前 store 的状态及提交修改操作。

``` javascript
// 页面组件中
import {mapActions} from 'vuex';

export default {
    name: 'detail',
    methods: {
        ...mapActions('appShell/appHeader', [
            'setAppHeader'
        ]),
        ...mapActions('appShell/appBottomNavigator', [
            'hideBottomNav'
        ])
    },
    created() {
        this.setAppHeader({
            show: true,
            title: 'Lavas',
            showMenu: false,
            showBack: true,
            showLogo: false,
            actions: [
                {
                    icon: 'home',
                    route: {
                        name: 'home'
                    }
                }
            ]
        });
        this.hideBottomNav();
    }
};
```

App Shell 和页面路由组件之间的通信是通过全局事件总线 EventBus 来实现的，具体实现中，App Shell 组件使用不同的命名空间触发事件，各页面路由组件在 activated 函数中进行监听，并处理自己的页面逻辑。

``` javascript
// App shell 组件中，发送全局事件，便于非父子关系的路由组件监听
EventBus.$emit(`app-header:click-menu`, eventData);
```

``` javascript
// 页面路由组件，在 activated 钩子中注册
activated() {
    EventBus.$on(`app-header:click-menu`, ({data}) => {
        // 处理点击按钮事件
        // ...
    });
}

```

关于这部分内容，在 [开发一个页面](https://lavas.baidu.com/guide/vue/doc/vue/01-foundation/03-how-to-add-a-page#与-app-shell-的交互) 中也有介绍。

App Shell 的组件在主页面 `src/App.vue` 中导入使用，如果要对 App Shell 进行扩展，需要在这里引用新增组件。

``` html
<!--> src/App.vue <-->
<template>
    <div id="app">
        <div class="app-shell">
            <app-header
                class="app-shell-header"
                @click-menu="handleClickHeaderMenu"
                @click-back="handleClickHeaderBack"
            >
                <template slot="logo"></template>
            </app-header>
        </div>
    </div>
</template>

<script>
import Vue from 'vue';
import {mapState, mapActions} from 'vuex';
import AppHeader from '@/components/AppHeader';

export default {
    name: 'app',
    components: {
        AppHeader
    }
};
</script>
```

## 小结

你可以结合 [Lavas Github](https://github.com/lavas-project) 给出的 [lavas-template-vue-appshell](https://github.com/lavas-project/lavas-template-vue-appshell) 更清晰地了解 Lavas 模板提供的 App Shell 结构。





