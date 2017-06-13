# 调整及扩展 app shell

开始之前，您可以查看 [App Shell](https://developers.google.com/web/fundamentals/architecture/app-shell?hl=zh-cn) 相关内容，快速掌握相关基础。

App Shell 就是一个简单的页面框架结构，在用户首屏渲染时，快速出现，避免甚至消除白屏时间过长，大大提升用户的体验。这里的简单一般是指不依赖js框架，同时能够很好的诠释页面的结构，一般仅包括了HTML片段、CSS样式及必要的图片等。能够离线缓存，当用户再次进入时，可重复使用缓存提升体验。

如果要开发一个App Shell, 首先需要明确区分页面 Shell 和 动态内容部分。一般而言，您的应用应加载尽可能最简单的 Shell，如在给出 bpwa-demo-news 中，我们将头部导航作为 Shell，其余部分为动态内容，也就需要适时更新的部分。明确了之后，我们就可以着手开发这部分了。

![示例](./images/app-shell-1.png.png)


## app-shells 管理和使用

导出项目中，提供了 `src/app-shells` 目录来统一存放 Shell， 单个 Shell 其实都是以一个组件的形式存在，只是考虑到其有自身的特殊性，我们将它独立出来管理。开发者可以根据需求自己定制 Shell 。每个 Shell 文件夹中包含一个 `.vue` 的主体文件、一个集中状态管理的容器 `store`（它包含了 Shell 所需的state、 actions、 mutations等）。如果您对 store 的概念还不是很熟悉，可以翻阅 [Vuex](https://vuex.vuejs.org/zh-cn/getting-started.html) 的状态管理模式文档，也可以参考[组件开发](./04-how-to-develop-a-component.md)。最终这里配置的 store 会在 `src/store` 中通过 Vuex 的 store 统一管理起来，在主页面 `src/App.vue` 中导入 Shell 组件即可使用。

``` html
<template>
    <div id="app">
        <app-shell></app-shell>
    </div>
</template>

<script>
import {AppShell} from '@/app-shells';

export default {
    name: 'app',
    components: {
        AppShell
    }
};
</script>
```



## app-shell store

每一个 Vuex 应用的核心就是 store（仓库）。`store`基本上就是一个容器，它包含着你的应用中大部分的状态(state)。所以我们在 Shell 或是组件开发时，都需要指定自己所需的store, 并通过 [Vuex](https://vuex.vuejs.org/zh-cn/getting-started.html)  来实现统一的管理。

``` js
// 将 store 统一集中管理
export default new Vuex.Store({
    getters: {},
    modules: {
        appShell, // 引入的 App Shell 的 store
        user,
        news,
        newsList
    }
});
```

`store` 和单纯的全局对象有以下两点不同：

* Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。

* 您不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交(commit) mutations。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。


再次强调，我们通过提交 mutation 的方式，而非直接改变 `store.state` 的变量，是因为我们想要更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易地解读应用内部的状态改变。此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。有了它，我们甚至可以实现如时间穿梭般的调试体验。

由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的 methods 中提交 mutations。


## 小结

大家结合官网给出的新闻 pwa-demo-news 可以加深对此处的理解。



