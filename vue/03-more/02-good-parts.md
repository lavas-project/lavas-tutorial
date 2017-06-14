# 最佳实践建议

开发 PWA 项目过程中，为了能够开发出高性能的代码，我们需要对自己的代码进行一系列的规范约定。在本篇教程中列出一些编写高性能可维护代码的实践建议，供大家参考。

## 代码规范

首先参照规范写出的代码在可维护性和代码性能方面是非常有效的方法，我们在这里推荐大家参照百度前端编码规范:

- JavaScript 代码规范： https://github.com/ecomfe/spec/blob/master/javascript-style-guide.md
- ES6 代码规范：https://github.com/ecomfe/spec/blob/master/es-next-style-guide.md
- CSS 代码规范：https://github.com/ecomfe/spec/blob/master/css-style-guide.md
- 更多代码规范：https://github.com/ecomfe/spec

针对百度前端代码规范的 lint 工具，推荐使用 [Fecs](http://fecs.baidu.com) 来验证代码规范。


## 命名规范

编码时，在命名上面开发者往往很容易忽视一些细节，由于我们的架构是 SPA 类型，最终代码都将打包到一起发布，如果命名发面发生冲突尤其是 HTML 和 样式的命名冲突可能导致意想不到的问题。

**class 名**

- 以中划线分割词义
- div 标签必须包含 class 属性
- 避免出现简单的 class 名 (`a`, `user`, `pwd` 等)
- class 名需要对功能表达明确 (如 `home-login-btn` 表示 home 页面的登录按钮)

**component 名**

- 定义 Vue 组件名时统一使用驼峰命名
- 自定义 Component 标签时统一使用中划线分割命名
- 表达意义要明确


## Vue 开发

通过对 vue 的学习，我们都知道在编写 Vue 代码的过程中需要慎用 dom 操作。

- 用户的操作行为统一通过 Component 事件或 Vuex action 处理
- action 内做的操作的只是通过调用 mutation 对 store 数据进行操作，不涉及任何 dom 操作
- 了解 Vue 生命周期，在不同的生命周期 hook method 中做恰当的操作
- 合理封装 Component，减少重复开发工作


## 业务代码实现原则

- 保证 Web App 可访问是最低准则。
- 以页面为粗粒度的按需加载静态资源。
- 以功能组件和业务组件为粒度来开发维护组件。
- 控制 service worker precache 的文件内容和数量 (CacheStorage 容量有限)。

