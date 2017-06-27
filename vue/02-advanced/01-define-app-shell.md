# App Shell 调整及扩展

开始之前，您可以查看 [App Shell](https://developers.google.com/web/fundamentals/architecture/app-shell?hl=zh-cn) 相关内容，快速掌握相关基础。

App Shell 就是一个简单的页面框架结构，在用户首屏渲染时，快速出现，避免甚至消除白屏时间过长，大大提升用户的体验。这里的简单一般是指不依赖js框架，同时能够很好的诠释页面的结构，一般仅包括了HTML片段、CSS样式及必要的图片等。能够离线缓存，当用户再次进入时，可重复使用缓存提升体验。

如果要开发一个 App Shell, 首先需要明确区分页面 Shell 和 动态内容部分。一般而言，您的应用应加载尽可能最简单的 Shell，如在给出 lavas-demo-news 中，我们将头部导航作为 Shell，其余部分为动态内容，也就需要适时更新的部分。明确了之后，我们就可以着手开发这部分了。

![示例](./images/app-shell-1.png)


## App Shell 管理和使用

导出的 webshell 项目中，提供了部分的 Shell 组件， 单个 Shell 其实都是由不同组件组成，组件在 src/components 中管理。开发者可以在 App.vue 中根据需求自己定制 Shell，也可以在我们提供的组件基础上进行二次开发。每个 Shell 所需的输入都在一个集中状态管理的容器 store 中统一管理，它包含了 Shell 所需的state、 actions、 mutations 等。 具体 Shell 配置可查看 src/store/modules/app-shell.js，在 src/store/index.js 中统一引入管理，在主页面 src/App.vue 中导入 Shell 组件并使用。如果您对 store 的概念还不是很熟悉，可以翻阅 [Vuex](https://vuex.vuejs.org/zh-cn/getting-started.html) 的状态管理模式文档。

``` html
<template>
    <div id="app">
        <div class="app-shell">
            <app-header
                class="app-shell-header"
                @click-menu="handleClickHeaderMenu"
                @click-back="handleClickHeaderBack">
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



## Vuex store

每一个 Vuex 应用的核心就是 store（仓库）。store 基本上就是一个容器，它包含着你的应用中大部分的状态(state)。所以我们在 Shell 或是组件开发时，都需要指定自己所需的 store, 并通过 [Vuex](https://vuex.vuejs.org/zh-cn/getting-started.html)  来实现统一的管理（src/store/index.js）。

``` js
// 将 store 统一集中管理
export default new Vuex.Store({
    getters: {},
    modules: {
        appShell, // 引入的 App Shell 的 store
        // user // 其他组件
    }
});
```

store 和单纯的全局对象有以下两点不同：

* Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。

* 您不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交(commit) mutations。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。


再次强调，我们通过提交 mutation 的方式，而非直接改变 store.state 的变量，是因为我们想要更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易地解读应用内部的状态改变。此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。有了它，我们甚至可以实现如时间穿梭般的调试体验。

由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的 methods 中提交 mutations。


## 小结

大家结合 Lavas Github(https://github.com/lavas-project) 给出的 lavas-template-vue-webshell 可以加深对此处的理解。



