## lavas static

使用 Lavas 内置的静态服务器启动__前端渲染__的 Lavas 项目。

比较常规的用法是在前端渲染的项目进行构建后使用此命令进行启动。具体来说，如果开发者的项目使用的是前端渲染 (`ssr: false`)，并且经过 `lavas build` 之后，在生成的 `/dist` 目录中，可以使用 `lavas static` 启动项目。在这种模式下，所有的代码均经过了 babel 转码和 webpack 压缩，并且 Service Worker 也会被注册，使用 localhost 访问即可预览效果。

![lavas-static](https://boscdn.baidu.com/assets/lavas/codelab/lavas-static.png)

因为启动的是静态服务器，限制较少功能也更加灵活。除了进行 Lavas 开发之外，也可以用作普通的静态服务器在__任何目录启动__。
