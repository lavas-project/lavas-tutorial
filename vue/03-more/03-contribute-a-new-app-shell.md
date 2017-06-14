# 贡献一个新的 app-shell


## app-shells

app-shells 是提供给 PWA 工程的 Shell，是可以优先被服务端渲染起到渐进式效果的部分，如果站点没有 NodeJs Server 的后端的架构，我们就不能基于 Vue 的 Server Side Render 来处理 Shell，我们同样也可以使用 Skeleton 方案进行一些类 Shell 的处理。

我们为了能够提供一整套的 app-shell， 所以在模版中对 app-shells 部分进行了整体封装，也就是我们的 app-shell 是一个独立的 Component 和 Vuex Store 集合对象。


模版 git 地址：[https://gitgub.com/searchfe/bpwa-template-vue](https://gitgub.com/searchfe/bpwa-template-vue)

欢迎 fork 代码，贡献 app-shell
