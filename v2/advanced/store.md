# Vuex 状态树

参考[ Nuxt Vuex store](https://zh.nuxtjs.org/guide/vuex-store) 的实现，Lavas 支持以模块方式组织 Vuex 的状态树。
对于开发者来说，只需要在项目根目录下 `/store` 文件夹中创建单独的模块文件，Lavas 会将这些单独的模块组合起来，生成最终的 `Vuex.Store` 实例。

下面我们以模板项目中已有的负责页面切换的 `pageTransition` 模块为例，介绍具体使用方法。如果您对 Vuex 尤其是[模块](https://vuex.vuejs.org/zh-cn/modules.html)特性还不了解，推荐查阅官方文档学习。

## 创建单独的模块文件

通常我们在使用 Vuex 定义状态时，会将 [state](https://vuex.vuejs.org/zh-cn/state.html) 和 [mutation](https://vuex.vuejs.org/zh-cn/mutations.html) 放在一个对象中传给 `Vuex.Store` 以创建最终的状态树，类似这样：
```javascript
const store = new Vuex.Store({
    state: {
        count: 1
    },
    mutations: {
        increment (state) {
            state.count++
        }
    }
})
```

在 Lavas 中定义状态和上述做法略有不同，我们以创建的模板项目为例，在 `/store` 目录下我们看到有一个单独的 `pageTransition.js` 文件。其中暴露了 `state` 和 `mutation`，当然也可以暴露包含异步操作的 [action](https://vuex.vuejs.org/zh-cn/actions.html)。Lavas 会以此创建一个名为 `pageTransition` 的模块。
```javascript
export const state = () => {
    return {
        type: 'none',
        effect: 'none'
    };
};

export const mutations = {
    setType(state, type) {
        state.type = type;
    },
    setEffect(state, effect) {
        state.effect = effect;
    }
};
```

> info
>
> 注意这里的 `state` 是一个返回初始状态的**方法**而非简单对象。这是由于在 SSR 场景下，单一状态会被多个服务端 Vue 实例共享，造成互相影响，因此需要每次调用 `state` 方法创建新的初始状态对象。如果您只是在 SPA/MPA 场景下使用 Lavas，可以不用遵守这个规则，直接返回初始状态对象即可。

## 在组件中使用

下面我们来看看如何在组件中访问定义好的状态。

我们以模板项目中 `/core/App.vue` 为例，由于创建了 `pageTransition` 这个命名空间，我们需要把模块的空间名称字符串，也就是 `pageTransition` 作为第一个参数传递给 `mapState` 函数，这样所有绑定都会自动将该模块作为上下文，使用 `mapActions` 同理。
```javascript
import {mapState} from 'vuex';

// 在模板中使用
<transition
    :name="pageTransitionEffect">

// 使用 mapState 绑定到 this 上
computed: {
    ...mapState('pageTransition', {
        pageTransitionEffect: state => state.effect
    })
}
```
