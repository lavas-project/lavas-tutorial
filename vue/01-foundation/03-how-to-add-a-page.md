# 开发一个页面

在模版项目中，所有的路由组件页面都放在`src/pages`下。我们以`NotFound`页面为例，介绍一下页面开发的基本步骤。

## 添加路由

模版项目使用异步懒加载路由对象的方式，减少首屏请求大小，所以需要在`router.js`中定义[代码切割点](https://ssr.vuejs.org/en/routing.html)。
然后向 vue-router 的路由列表中添加页面对应的路由对象，代码如下：

```js
const NotFound = () => import('@/pages/NotFound.vue');

routes: [
    // <!-- INJECT_SKELETON_ROUTE -->
    // 省略其他路由对象
    {
        path: '*',
        name: 'notFound',
        component: NotFound
    }
]
```

这里有两点需要注意：
1. `// <!-- INJECT_SKELETON_ROUTE -->`这行占位符不能删除，在开发环境下作为[skeleton 路由](/guide/vue/doc/vue/01-foundation/14-skeleton)的插入点。
2. 路由组件默认使用了 [keep-alive](https://vuejs.org/v2/guide/components.html#keep-alive) ，避免切换时重新渲染。如果不想使用，可以通过路由对象的`meta.notKeepAlive`属性关闭。
    ```js
        {
            path: '/',
            name: 'home',
            component: Home,
            meta: {
                notKeepAlive: true
            }
        }
    ```

## 页面组织结构

还是以`NotFound`页面为例，一个典型的`.vue`单文件组件包含`template`，`script`和`style`三部分：
``` vue
    <template>
    </template>

    <script>
    import {mapActions} from 'vuex';
    import pageLoadingMixin from '@/mixins/pageLoadingMixin';

    export default {
        name: 'notFound',
        mixins: [pageLoadingMixin],
        methods: {
            ...mapActions([
                'setPageLoading',
                'showBottomNav',
                'setAppHeader'
            ])
        },
        activated() {
            this.setAppHeader({});
            this.hideBottomNav();
            this.setPageLoading(false);
        }
    };
    </script>

    <style lang="stylus" scoped>
    </style>
```

## 与 app shell 的交互

每个页面都需要与 app shell 进行交互，例如改变头部导航条右侧的图标，设置当前页面标题，展示载入中状态等。

### 通过 vuex 提交修改动作

模版项目中 app shell 组件的状态放在 store 中统一管理，页面组件可以通过`mapGetters/Actions`访问当前 store 的状态和提交修改操作。

在具体实现中，`app-shells/BottomNavigation/store/index.js`中的`actions`对象定义了一系列操作，通过`mapActions`引入就可以在组件中使用这些方法，代码如下：

```js
const actions = {
    /**
     * 设置顶部导航条
     */
    setAppHeader({commit}, appHeader) {
        commit(types.SET_APP_HEADER, appHeader);
    },
    /**
     * 隐藏底部导航
     */
    hideBottomNav({commit}) {
        commit(types.SET_APP_BOTTOM_NAV, {show: false});
    },
    ...
}
```

那么在路由组件中，何时调用这些操作方法呢？前面介绍过，路由组件默认使用了 keep-alive，会在生命周期中增加两个钩子函数：`activated`和`deactivated`，分别在组件激活和注销时触发。在`NotFound`页面中，在`activated`时进行了设置头部，隐藏底部导航条等操作。

如果路由组件禁止了 keep-alive，可以在例如`mounted`事件钩子中触发以上操作。

### 监听全局事件

除了直接调用操作方法修改 app shell 的状态，路由组件还可以监听 app shell 组件发出的全局事件。在具体实现中，由于 app shell 和路由组件并不存在父子关系，所以事件会通过全局事件总线 EventBus 进行发送和接收。

app shell组件触发的事件如下，为了避免重复，在事件名之前都加上了命名空间：
* AppHeader 顶部导航条
    * app-header:click-menu 点击左侧菜单图标
    * app-header:click-back 点击左侧返回图标
    * app-header:click-logo 点击Logo图标
    * app-header:click-action 点击右侧动作图标，事件对象中包含当前动作序号
* AppBottomNavigator 底部导航条
    * app-bottom-navigator:click-nav 点击底部项目，事件对象中包含当前导航项目`name`

当路由组件想监听事件时，只需要在`activated`钩子中注册事件处理函数：
```js
    import EventBus from '@/event-bus';
    // 在activated钩子中注册
    EventBus.$on(`app-header:click-action`, ({actionIdx}) => {
        // 处理点击按钮事件
    });
```

### 加载中动画展示

app shell 中还包含了全局的加载中动画，在页面切换时显示，路由组件在合适的时机隐藏。

加载中以[mixin](https://vuejs.org/v2/guide/mixins.html)的形式，位于`mixins/pageLoadingMixin`中，在离开路由时开启，关闭的时机由路由组件决定，例如`NotFound`页面不需要异步加载数据，所以在`activated`钩子中关闭加载中动画。

```js
    beforeRouteLeave(to, from, next) {
        // 离开组件对应的路由时，开启loading
        this.setPageLoading(true);
        next();
    }
```

全局的加载中并不一定适合所有场景，例如无限滚动上拉加载更多，或者多 tab 切换加载数据。这些加载中的效果需要在具体组件中实现。

## 组件开发

[vuetify](https://vuetifyjs.com)提供了丰富的组件。

开发自己的业务组件时，可参考[官方的组件开发指南](https://vuejs.org/v2/guide/components.html)。

## 异步请求数据

在 vuex 中，由于 mutation 必须是同步函数，异步请求可以放在 action 中执行，通过使用[async/await新特性](https://vuex.vuejs.org/zh-cn/actions.html)使代码变的简洁。

推荐使用[axios](https://github.com/mzabriskie/axios)与服务端通信。axios 基于 Promise，兼容浏览器端和node.js环境。




