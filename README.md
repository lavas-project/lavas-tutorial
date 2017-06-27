<style lang="stylus" scoped="off">
    .md-content
        padding 0
</style>

<style lang="stylus">
    $color-black = #263238
    $color-blue = rgb(40, 116, 240)
    $color-white = #fff
    $color-grey = #ccc
    $color-grey-light = #f0f5f6
    $color-green = #4caf50

    .to-github
        display none

    .m-container
        width 100%

    .m-box
        max-width 1200px
        overflow hidden
        margin 0 auto
        padding 0 20px

    .m-title
        padding 0
        margin-bottom 20px
        border-left 4px solid $color-blue
        padding-left 14px
        padding-bottom 0
        line-height 1

    .m-def
        color $color-white
        text-align center
        overflow hidden
        padding 60px 0 40px

        h1
            color $color-white
            font-size 48px
            margin-bottom 20px

        p
            margin-bottom 40px

        .btn-box
            text-align center


        .m-btn
            display inline-block

    .m-intro
        max-width 1200px
        overflow hidden
        margin 0 auto
        padding-bottom 20px

    .m-intro-box
        width 33.33%
        float left
        padding 0 20px 0 0
        box-sizing border-box

        h2
            text-align center

    .m-btn
        display block
        position relative
        width 200px
        height 40px
        line-height 40px
        border-radius 3px
        text-align center
        margin 0 auto
        font-weight 600
        box-shadow 0 1px 5px rgba(0, 0, 0, .2), 0 2px 2px rgba(0, 0, 0, .14), 0 3px 1px -2px rgba(0, 0, 0, .12)

        &:hover
            text-decoration none

            &:after
                content ''
                position absolute
                top 0
                right 0
                bottom 0
                left 0
                background $color-white
                opacity .12
                border-radius inherit

        &:active
            box-shadow 0 5px 5px -3px rgba(0, 0, 0, .2), 0 8px 10px 1px rgba(0, 0, 0, .14), 0 3px 14px 2px rgba(0, 0, 0, .12)

        &.m-blue
            color $color-white

        &.m-grey
            color $color-black
            background $color-grey
            &:hover
                &:after
                    background $color-white

        &.m-green
            color $color-white
            background $color-green

        &.m-white
            color $color-black
            background $color-white
            &:hover
                &:after
                    background $color-black

    .m-blue
        background $color-blue !important

    .m-grey-light
        background $color-grey-light

    .m-tool
        padding-bottom 40px
    .m-code-box
        overflow hidden
    .m-code
        float left
        width 50%
        box-sizing border-box
        h2
            margin 20px 0 10px
        &:nth-child(odd)
            padding-right 15px
        &:nth-child(even)
            padding-left 15px
        pre
            padding 12px
            code
                margin 0
        h3
            margin-bottom 10px

    @media screen and (min-width: 1001px)
        .m-intro-box
            p
                height 90px
    @media screen and (max-width: 1000px)
        .m-code
            float none
            width 100%
            padding 0 !important

    @media screen and (max-width: 900px)
        .m-intro-box
            h2
                height 48px

            p
                height 110px


    @media screen and (max-width: 800px)
        .m-intro-box
            width 100%
            padding 0

            h2
                height auto

            p
                height auto

    @media screen and (max-width: 600px)
        .m-def
            .btn-box
                .m-btn
                    display block
                    margin-bottom 10px

</style>

<div class="m-container m-blue">
    <div class="m-box m-def">
        <h1>Lavas</h1>
        <p>基于 Vue 的 PWA 解决方案，帮助开发者快速搭建 PWA 应用，解决接入 PWA 的各种问题。</p>
        <div class="btn-box">
            <a class="m-btn m-white" href="/guide/vue/doc/vue/01-foundation/02-quick-tour-by-cli">起步</a>
            <a class="m-btn m-grey" target="_blank" href="https://github.com/lavas-project/lavas-cli">Github</a>
        </div>
    </div>
</div>
<div class="m-box">
    <h1 class="m-title">开始吧</h1>
    <p>通过本教程，您将很快可以创建一个属于您的 Lavas PWA 框架。</p>
</div>
<div class="m-intro m-box">
    <div class="m-intro-box">
        <h2>开始探索 Lavas 框架</h2>
        <p>了解 Lavas 框架提供了什么功能，以及如何使用框架进行快速开发</p>
        <a class="m-btn m-blue" href="/guide/vue/doc/vue/01-foundation/01-get-start">开始探索</a>
    </div>
    <div class="m-intro-box">
        <h2>创建 Lavas PWA 框架</h2>
        <p>立即开始使用最新版本的框架进行开发吧。您可以选择在线服务或者使用命令行工具创建 Lavas PWA 框架哦。</p>
        <a class="m-btn m-blue" href="/guide/vue/start">在线创建</a>
    </div>
    <div class="m-intro-box">
        <h2>了解 PWA</h2>
        <p>学习 PWA，了解这项新技术将会给您的项目带来的收益。</p>
        <a class="m-btn m-blue" href="/doc">点击了解</a>
    </div>
</div>

<div class="m-container m-grey-light">
    <div class="m-box m-tool">
        <h1 class="m-title">获取工具</h1>
        <p>
            <code>lavas-cli</code> 是一个命令行工具，可以通过它来快速创建 PWA 项目。它提供了多种常用的 APP Shell 给用户选择，降低了开发成本，也简化了工作流程，让 PWA 项目变得易于管理。
        </p>
        <p>可以通过 <code>npm install</code> 方便快捷地安装 lavas-cli：</p>
        <div class="m-code-box">
            <div class="m-code">
                <h2>安装</h2>
                <pre><code>[sudo] <span class="hljs-built_in">npm</span> install -g <span class="hljs-keyword">lavas-cli</span></code></pre>
            </div>
            <div class="m-code">
                <h2>使用</h2>
                <pre><code><span class="hljs-keyword">lavas-cli</span> init</code></pre>
            </div>
        </div>
    </div>
</div>
