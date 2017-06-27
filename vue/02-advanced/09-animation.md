# 动画介绍

Vue 提供了多种方式支持动画过渡效果。例如在各个过渡阶段应用 CSS 类，提供钩子函数使用 JS 操作 DOM，使用第三方 CSS/JS 动画库等。

如果对 Vue 中内置的 transition 机制还不了解，可以阅读[官方的介绍](https://vuejs.org/v2/guide/transitions.html)。

在模版项目中，主要使用了最简单的应用 CSS 类的方式完成动画效果。

## 页面切换动画效果

在模版项目中，页面切换时，会有左右滑动效果。
具体表现为打开新页面时左滑展示，返回之前的页面时右滑退出。
下面简单介绍下实现原理。

在 BottomNavigation 中，`<router-view>`对应的每一个路由页面组件在 DOM 中添加 ／ 移除时，都会被应用 pageTransitionName 对应的 CSS 类，在左/右滑对应的类名为 slide-left/right ：

```html
<transition
    :name="pageTransitionName">
    <router-view></router-view>
</transition>
```

以左滑为例，即将打开的新页面插入 DOM 时立即应用 slide-left-enter 将页面移动到屏幕右外侧，随即移除，页面表现为向左滑回到初始位置。
而旧页面在整个离开过程中都应用 slide-left-leave-active，也向左滑动至屏幕左外侧。

```css
.app-view
    &.slide-left-enter
        transform translate(100%, 0)

    &.slide-left-leave-active
        transform translate(-100%, 0)
```

关于 v-enter，v-leave-active 这些 CSS 类的添加时机，可参阅[文档 Transition Classes 一节](https://vuejs.org/v2/guide/transitions.html#Transition-Classes)。

如果需要更换页面切换效果，例如想使用渐隐/渐显代替滑动，只需要简单修改上述 CSS 类的样式规则即可：

```css
.app-view
    opacity 1
    transition opacity 1s ease

    &.slide-left-enter
        opacity 0

    &.slide-left-leave-active
        opacity 0
```

> info
>
> 如果不需要页面的切换动画，可以在 appShell 的状态树中通过设置 needPageTransition 进行关闭。
