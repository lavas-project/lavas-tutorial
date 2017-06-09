<style lang="stylus" scoped="off">
    .markdown-wrapper
        margin-right 0 !important
</style>
<style lang="stylus">
    $colorPrimaryBlue = rgb(40, 116, 240)
    $colorBlack = #263238

    .get-start
        color $colorBlack

    .title-wrapper
        margin 0 0 20px
        padding 50px 0 25px
        border-bottom 1px solid #eaecef

        h1
            border none
            margin 0
            font-weight 600
            font-size 32px

        p
            font-size 16px
            margin 0

    .fast-link-wrapper
        overflow hidden
        padding-bottom 20px

    .fast-link-item
        width 33.33%
        box-sizing border-box
        float left
        padding 0 10px

        h3
            text-align center
            font-weight 600
            color $colorBlack

        .md-a-btn
            width auto

        p
            height 120px

    h2
        font-weight 500
        color $colorBlack

    .md-a-btn
        color #fff
        display block
        width 200px
        height 40px
        line-height 40px
        border-radius 3px
        background $colorPrimaryBlue
        margin 0 auto
        text-align center
        position relative
        box-shadow 0 1px 5px rgba(0,0,0,.2), 0 2px 2px rgba(0,0,0,.14), 0 3px 1px -2px rgba(0,0,0,.12)
        &:hover
            text-decoration none
            &:after
                content ''
                position absolute
                top 0
                right 0
                bottom 0
                left 0
                background #fff
                opacity .12
                border-radius inherit
        &:active
            box-shadow 0 5px 5px -3px rgba(0,0,0,.2), 0 8px 10px 1px rgba(0,0,0,.14), 0 3px 14px 2px rgba(0,0,0,.12)
    .md-a-btn,
    .md-a-btn:after
        -webkit-transition .3s cubic-bezier(.25,.8,.25,1)
        transition .3s cubic-bezier(.25,.8,.25,1)

    @media screen and (max-width: 900px)
        .get-start

            .fast-link-item
                h3
                    height 48px

                p
                    height 140px


    @media screen and (max-width: 800px)
        .get-start

            .fast-link-item
                width 100%
                h3
                    height auto

                p
                    height auto
</style>

<div class="get-start">
    <div class="title-wrapper">
        <h1>开始吧</h1>
        <p>通过本教程，您将很快可以创建一个属于您的 PWA 框架。</p>
    </div>
    <div class="fast-link-wrapper">
        <div class="fast-link-item">
            <h3>开始探索 PWA 工程</h3>
            <p>了解 PWA 框架提供了什么功能，以及如何使用框架进行快速开发。</p>
            <a href="/guide/vue/doc/vue/01-foundation/01-get-start" class="md-a-btn">开始探索</a>
        </div>
        <div class="fast-link-item">
            <h3>创建 PWA 框架</h3>
            <p>立即开始使用最新版本的框架进行开发吧。您可以选择在线服务或者使用命令行工具创建 PWA 框架哦。</p>
            <a href="/guide/vue/start" class="md-a-btn">在线创建</a>
        </div>
        <div class="fast-link-item">
            <h3>了解 PWA</h3>
            <p>学习 PWA，了解这项新技术将会给您的项目带来的收益。</p>
            <a href="/doc" class="md-a-btn">点击了解</a>
        </div>
    </div>
    <div class="block-wrapper">
        <h2>获取工具</h2>
    </div>
</div>

`bpwa-cli` 是一个命令行工具，可以通过它来快速创建 PWA 项目。它提供了多种常用的 APP Shell 给用户选择，降低了开发成本，也简化了工作流程，让 PWA 项目变得易于管理。

可以通过 `npm install` 方便快捷地安装 bpwa-cli：

```bash
[sudo] npm install -g bpwa-cli
bpwa init
```

想了解更多关于 bpwa-cli 的使用方法，请戳下面的链接：

<div>
    <a href="/guide/vue/doc/vue/01-foundation/02-quick-tour-by-cli"
        class="md-a-btn"
    >安装 bpwa-cli</a>
</div>