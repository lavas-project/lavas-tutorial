# 检验 PWA 站点


## CheckList

对于 PWA 站点是否完善，Google 通过 PWA 所需要具备的一些特性和最佳实践给出了一个 Checklist, 该 Checklist 从多个方面来提供 PWA 站点检测的标准。如果我们对 PWA 工程的检测要求集成到持续集成系统的话(类似后面所提到的 Lighthouse)，我们可以参照 Checklist 进行一系列的检测。

Checklist: [https://developers.google.cn/web/progressive-web-apps/checklist](https://developers.google.cn/web/progressive-web-apps/checklist)

## Chrome Devtool

Chrome 有一个强大之处在于它的 Devtool 的强大，我们可以通过 Chrome Devtool 进行一系列的检测和验证自己的 PWA 站点。

**进入方式**

- windows: `F12`
- mac: `Command + Option + I`

**NetWork**

我们可以通过 Network 选项卡观察 Web App 的网络请求情况。
可以观察某一个静态资源或者异步请求的请求情况、缓存情况、返回情况等。

![chrome network](./images/chrome-network.png)

**Application**

调试检测 PWA 项目， Application 选项卡主要是用来调试和观察 Service Worker 文件以及 CacheStorage 缓存的。

![chrome application](./images/chrome-application.png)

对于 PWA 工程的具体调试，可以参考 [Service Worker 调试](https://lavas.baidu.com/doc/offline-and-cache-loading/service-worker/04-service-worker-debug)
也可以深入了解 [Google Chrome Devtool](https://developers.google.cn/web/tools/chrome-devtools)


## Lighthouse

Lighthouse 是 Google 开发的一个检验站点性能相关的一个应用。您可以将其作为一个 Chrome 扩展程序运行，或从命令行运行。 您为 Lighthouse 提供一个您要审查的网址，它将针对此页面运行一连串的测试，然后生成一个有关页面性能的报告。

通过 Lighthouse ，您可以从失败的点入手，看看可以采取哪些措施来改进您的应用。

注：Lighthouse 目前非常关注 PWA 功能，如`添加到主屏幕`和离线支持。不过，此项目的首要目标是针对网络应用质量的各个方面提供端到端审查。


运行 Lighthouse 的方式有两种：作为 Chrome 扩展程序运行，或作为命令行工具运行。 Chrome 扩展程序提供了一个对用户更友好的界面，方便读取报告。 命令行工具允许您将 Lighthouse 集成到持续集成系统。

详细信息，可以去 [Lighthouse 官方文档](https://developers.google.cn/web/tools/lighthouse) 查看


由于 Lighthouse Chrome 插件由于某些墙的原因在国内无法下载。我们提供一套可以在国内安装插件的步骤如下：

**1、下载 lighthouse 插件**

Lighthouse 浏览器插件 [下载地址](./downloads/lighthouse_2.1.0_0.zip)

**2、将下载的 zip 包解压缩到某一个目录下**

假设存放至 `/home/work/chrome/extensions`

**3、在 Chome 地址栏输入 `chome://extentions` 打开浏览器应用程序**

点击 `开发者模式` 进入开发者模式，点击 `加载已解压的扩展程序` 选择第 2 步下载的 zip 包解压的文件夹

**4、安装成功**

安装成功后可以看到浏览器右上角出现 Lighthouse 图标。

- 通过 Chrome 访问任何页面。
- 在访问的页面地址栏的右上角上点击 Lighthouse 插件的按钮。
- 点击 `Generate report`。
- 坐等出检测报告。
- 根据报告的内容查看当前站点的优缺点，进而优化。


## 真机测试

- 手机百度 / 百度手机浏览器
- Chrome / fireFox 浏览器
- 主流的安卓机型的自带浏览器
- 微信 / QQ 浏览器
- UC 浏览器
- 360 浏览器
- 猎豹浏览器
