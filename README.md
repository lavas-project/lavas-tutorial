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

.md-related-wrapper
    display none

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
    padding-top 60px
    padding-bottom 40px

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
    padding-bottom 30px

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
    overflow hidden
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

    &:active
        box-shadow 0 5px 5px -3px rgba(0, 0, 0, .2), 0 8px 10px 1px rgba(0, 0, 0, .14), 0 3px 14px 2px rgba(0, 0, 0, .12)

    &.m-blue
        color $color-white

    &.m-grey
        color $color-black
        background $color-grey

    &.m-green
        color $color-white
        background $color-green

    &.m-white
        color $color-black
        background $color-white

.icon-svg
    width 16px
    height 16px
    vertical-align middle
    fill $color-black
    margin-top -3px

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
        <p>基于 Vue 的 PWA 解决方案，帮助开发者快速搭建 PWA 应用，解决接入 PWA 的各种问题</p>
        <div class="btn-box">
            <a class="m-btn m-white" href="/guide/vue/doc/vue/01-foundation/02-quick-tour-by-cli">
                <div class="md-ripple"></div>
                <svg t="1498645229475" class="icon-svg" style="" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="3493" xmlns:xlink="http://www.w3.org/1999/xlink" width="200" height="200">
                    <path d="M951.975936 107.853824c-2.75968-2.184192-6.536192-2.57024-9.614336-0.944128L74.161152 550.883328c-3.231744 1.659904-5.136384 5.085184-4.855808 8.702976 0.318464 3.615744 2.761728 6.690816 6.221824 7.794688l290.622464 92.377088c3.057664 0.959488 6.360064 0.242688 8.704-1.88928L776.901632 295.631872 455.88992 675.607552c-1.941504 2.28864-2.604032 5.41696-1.747968 8.29952 0.854016 2.884608 3.09248 5.157888 5.971968 6.030336l322.486272 98.23232c0.833536 0.263168 1.714176 0.384 2.5856 0.384 1.533952 0 3.072-0.402432 4.438016-1.17248 2.12992-1.220608 3.6864-3.265536 4.261888-5.661696L955.117568 116.992C955.943936 113.586176 954.715136 110.021632 951.975936 107.853824z" p-id="3494"></path>
                    <path d="M554.471424 760.485888l-89.129984-27.4176c-2.692096-0.874496-5.659648-0.349184-7.933952 1.343488-2.289664 1.698816-3.636224 4.370432-3.636224 7.201792l0 168.221696c0 3.986432 2.624512 7.48032 6.464512 8.597504 0.805888 0.228352 1.645568 0.350208 2.48832 0.350208 3.004416 0 5.885952-1.52064 7.549952-4.160512l89.119744-140.798976c1.504256-2.361344 1.819648-5.279744 0.841728-7.882752C559.273984 763.319296 557.140992 761.309184 554.471424 760.485888z" p-id="3495"></path>
                </svg>
                起步
            </a>
            &nbsp;
            <a class="m-btn m-grey" target="_blank" href="https://github.com/lavas-project">
                <div class="md-ripple"></div>
                <svg t="1498641781602" class="icon-svg" style="" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg" p-id="2360" xmlns:xlink="http://www.w3.org/1999/xlink" width="200" height="200">
                    <path d="M950.857143 512q0 143.428571-83.714286 258t-216.285714 158.571429q-15.428571 2.857143-22.571429-4t-7.142857-17.142857l0-120.571429q0-55.428571-29.714286-81.142857 32.571429-3.428571 58.571429-10.285714t53.714286-22.285714 46.285714-38 30.285714-60 11.714286-86q0-69.142857-45.142857-117.714286 21.142857-52-4.571429-116.571429-16-5.142857-46.285714 6.285714t-52.571429 25.142857l-21.714286 13.714286q-53.142857-14.857143-109.714286-14.857143t-109.714286 14.857143q-9.142857-6.285714-24.285714-15.428571t-47.714286-22-49.142857-7.714286q-25.142857 64.571429-4 116.571429-45.142857 48.571429-45.142857 117.714286 0 48.571429 11.714286 85.714286t30 60 46 38.285714 53.714286 22.285714 58.571429 10.285714q-22.857143 20.571429-28 58.857143-12 5.714286-25.714286 8.571429t-32.571429 2.857143-37.428571-12.285714-31.714286-35.714286q-10.857143-18.285714-27.714286-29.714286t-28.285714-13.714286l-11.428571-1.714286q-12 0-16.571429 2.571429t-2.857143 6.571429 5.142857 8 7.428571 6.857143l4 2.857143q12.571429 5.714286 24.857143 21.714286t18 29.142857l5.714286 13.142857q7.428571 21.714286 25.142857 35.142857t38.285714 17.142857 39.714286 4 31.714286-2l13.142857-2.285714q0 21.714286 2.857143 50.857143t2.857143 30.857143q0 10.285714-7.428571 17.142857t-22.857143 4q-132.571429-44-216.285714-158.571429t-83.714286-258q0-119.428571 58.857143-220.285714t159.714286-159.714286 220.285714-58.857143 220.285714 58.857143 159.714286 159.714286 58.857143 220.285714z" p-id="2361"></path>
                </svg>
                Github
            </a>
        </div>
    </div>
</div>
<div class="m-box">
    <h1 class="m-title">开始吧</h1>
    <p>通过本教程，您将很快可以创建一个属于您的 PWA 框架</p>
</div>
<div class="m-intro m-box">
    <div class="m-intro-box">
        <h2>探索 Lavas</h2>
        <p>了解 Lavas 解决方案提供了什么功能，学习如何使用命令行工具生成的框架进行快速开发</p>
        <a class="m-btn m-blue" href="/guide/vue/doc/vue/01-foundation/00-lavas-start"><div class="md-ripple"></div>开始探索</a>
    </div>
    <div class="m-intro-box">
        <h2>了解 PWA</h2>
        <p>学习 PWA，了解这项技术的创新点和要解决的问题，以及将会给您的项目带来的收益</p>
        <a class="m-btn m-blue" href="/doc"><div class="md-ripple"></div>点击了解</a>
    </div>
    <div class="m-intro-box">
        <h2>查看示例</h2>
        <p>基于 Lavas 命令行工具生成框架所开发的项目示例，直观感受 Lavas 的优点</p>
        <a class="m-btn m-blue" href="/demo"><div class="md-ripple"></div>查看示例</a>
    </div>
</div>

<div class="m-container m-grey-light">
    <div class="m-box m-tool">
        <h1 class="m-title">获取工具</h1>
        <p>
            <code>lavas</code> 提供了一个命令行工具，可以通过它来快速创建 PWA 项目。它提供了多种常用的 APP Shell 给用户选择，降低了开发成本，也简化了工作流程，让 PWA 项目变得易于管理。
        </p>
        <p>可以通过 <code>npm install</code> 方便快捷地安装 lavas：</p>
        <div class="m-code-box">
            <div class="m-code">
                <h2>安装</h2>
                <pre><code><span class="hljs-meta">$</span> <span class="hljs-built_in">npm</span> install -g <span class="hljs-keyword">lavas</span></code></pre>
            </div>
            <div class="m-code">
                <h2>使用</h2>
                <pre><code><span class="hljs-meta">$</span> <span class="hljs-keyword">lavas</span> init</code></pre>
            </div>
        </div>
    </div>
</div>
