# Lavas 命令介绍

开发者如果已经尝试过 Codelab 中“开发第一个 Lavas 应用”的[环境准备](/codelab/get-started/prepare)步骤，相信对于命令 `lavas init` 有所了解了。包括这条命令在内，这里将向开发者介绍 Lavas 集成的所有便捷命令。

## lavas init

同样在 Codelab 已经提过，使用 `lavas init` 命令进行项目的初始化，效果如下：

![lava init](./images/lavas-init.png)

## lavas build

使用 Lavas 对项目进行构建，内部会调用 babel, webpack 等，最终生成在 `/dist/` 目录中 (可以通过 `/lavas.config.js` 进行修改)，效果如下：

![lavas-build](./images/lavas-build.png)

更多关于构建的信息可以参见[构建部署工程](/guide/v2/basic/build)和[Lavas 中的 build 配置](/guide/v2/advanced/build)

## lavas dev

使用 Lavas 内置的调试服务器启动 Lavas 项目，方便开发者进行调试。调试服务器还包含了热加载 (hotreload) 功能，修改绝大部分的文件(如 `/pages/`, `/store/` 等)均__不__需要重启服务器，效果如下：

![lavas-dev](./images/lavas-dev.png)

此外为了防止 Service Worker 的缓存对频繁改动的开发调试阶段产生影响，使用 `lavas dev` 启动的调试服务器__不会__注册 Service Worker。

## lavas start

使用 Lavas 内置的正式服务器启动__服务端渲染__的 Lavas 项目。一般来说开发者在运行了 `lavas build` 之后，切换到生成目录(默认是 `/dist/`)中使用命令 `lavas start` 预览线上效果。在这种模式下，所有的代码均经过了 babel 转码和 webpack 压缩，并且 Service Worker 也会被注册，使用 localhost 访问即可预览效果。

![lavas-start](./images/lavas-start.png)

关于如何在正式环境上线 Lavas 项目还可以参见[构建部署工程](/guide/v2/basic/build)
