# 修改项目主题

> [Lavas Basic 模版](https://github.com/lavas-project/lavas-template-vue-basic)并不包含此功能

项目使用 [vuetify](https://vuetifyjs.com/) 作为组件库，根据 vuetify 的实现，主题配置分为两个部分：
1. 项目中主要使用的颜色，其中主要颜色，次要颜色的选择和使用场景可参考 [material 设计中的颜色系统](https://material.io/guidelines/style/color.html#color-color-system)。
2. [material 设计中使用的常量](https://material.io/guidelines/style/color.html#color-themes)包含了通用的背景色，前景色，文字颜色。

开发者在配置文件中定义变量，在开发自身控件时使用，使得更换主题变得非常方便。

## 配置文件

主题相关的配置文件在 `config/theme.js` 中，结构如下：

``` js

// 定义主题列表
const themeList = {

    // 定义主题名称
    myTheme: {
        themeColor: {
            primary: '#4DBA87',
            accent: '$blue.lighten-2',
            secondary: '$grey.darken-3',
            info: '$blue.base',
            warning: '$amber.base',
            error: '$red.accent-2',
            success: '$green.base'
        },
        materialDesign: {
            'bg-color': '#fff',
            'fg-color': '#000',
            'text-color': '#000',
            'primary-text-percent': .87,
            'secondary-text-percent': .54,
            'disabledORhints-text-percent': .38,
            'divider-percent': .12,
            'active-icon-percent': .54,
            'inactive-icon-percent': .38
        }
    }
};

module.exports = {
    theme: themeList.myTheme // 和主题列表中的主题名称对应
};
```

在 themeList 里可以定义多个主题，每个主题包含两部分：

* themeColor

    * `primary` 主要颜色，vuetify 中大部分组件以及我们的 app shell 都会大量使用

    * `accent` 通常根据主要颜色调整明暗后生成，以显示层次感

    * `secondary` 次要颜色，通常与主要颜色形成对比

    * `info/warning/error/success` 项目不同状态下定义的颜色，例如 vuetify 在 [Alerts](https://vuetifyjs.com/components/alerts) 组件中会使用
    
* materialDesign 包含了一系列 material 设计相关的变量

> info
>
> 以上变量都有默认值，所以开发者无需定义每一个变量，通常只需要关心主要颜色和次要颜色即可。

### 使用预定义的颜色变量

在上面的配置文件示例中使用到了 `accent: '$blue.lighten-2'` 来定义颜色，原因是 vuetify 预先定义了一系列 [stylus 颜色变量](https://vuetifyjs.com/style/colors)。

![vuetify 定义的颜色变量](./images/vuetify-color.png)

## 使用主题变量

在开发自己的控件时，需要尽量关联以上主题变量。

在具体使用时，不需要在每一个 `.vue` 文件的样式部分使用 @import 来引入这些变量，直接使用即可，方法如下：
* 使用 themeColor 中的变量：`background: $theme.primary`
* 使用 materialDesign 中的变量：`color: $material-theme.bg-color`

> info
>
> 需要注意的是，使用主题相关的变量时，`:` 不能够省略，原因是[省略冒号的情况下，stylus 编译器无法区分 hash 和选择器](https://github.com/stylus/stylus/issues/1405)


