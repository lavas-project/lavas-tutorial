# Lavas 安装

## 环境要求

* 一台可以正常联网的计算机并已安装较新版本的 Node.js (≥ 5.0) 和 npm (≥ 3.0)
* 一个方便调试并支持 Service Worker 的浏览器，推荐使用 Google Chrome
* 一个自己习惯的文本编辑器，如 Sublime Text, Web Storm 等等

## 安装步骤

1. 安装/升级 Lavas 命令行工具

    ```bash
    npm install lavas -g
    ```

2. 在合适的目录新建项目并命名，例如 "lavas-2-sample"

    ```bash
    lavas init
    ```

    ![lava init](https://boscdn.baidu.com/assets/lavas/codelab/lavas-init-3.png)

3. 进入目录并安装依赖

    ```bash
    npm install
    ```

4. 启动开发服务器，访问 localhost:3000 将看到初始页面。
    ```bash
    lavas dev
    ```
