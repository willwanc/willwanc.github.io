---
title: nuxt项目总结
tags: ['nuxt项目']
categories: ['nuxt']
date: 2022-04-06 13:34:41
---

## 目录结构

### `assets`

assets目录包含未经编译的资源。例如：stylus、less、sass、images或者fonts。

### `components`

组件目录包含应用公共 vue 组件。Nuxt.js 不会扩展增强该目录下 Vue.js 组件，**即这些组件不会像页面组件那样有 asyncData 方法的特性**。

### `layouts`

布局目录包含应用的布局组件。

### `middleware`

中间件允许您定义在呈现一个页面或一组页面（布局）之前运行的自定义函数。middleware 目录用于存放应用的共享中间件。文件名将是中间件的名称（middleware/auth.js 将是 auth 中间件）。您还可以通过直接使用函数来定义特定于页面的中间件。

### `pages`

页面目录 pages 用于组织应用的路由及视图。Nuxt.js 框架读取该目录下所有的 .vue 文件并自动生成对应的路由配置。**若无额外配置，该目录不能被重命名。**

### `plugins`

插件目录 plugins 用于组织那些需要在 根vue.js应用 实例化之前需要运行的 Javascript 插件。

### `static`

静态文件目录 static 用于存放应用的静态文件，此类文件不会被 Nuxt.js 调用 Webpack 进行构建编译处理。服务器启动的时候，该目录下的文件会映射至应用的根路径 / 下。

### `store`

store 目录用于组织应用的 Vuex 状态树 文件。 Nuxt.js 框架集成了 Vuex 状态树 的相关功能配置，在 store 目录下创建一个 index.js 文件可激活这些配置。

*若无额外配置，该目录不能被重命名。*

### nuxt.config.js 文件

nuxt.config.js 文件用于组织 Nuxt.js 应用的个性化配置，以便覆盖默认配置。

[关于 nuxt.config.js 的更多信息](https://nuxtjs.org/docs/configuration-glossary/configuration-alias)

#### 常用配置

- head: head标签配置。例如：title, meta, link等

- alias: 目录别名。例如按需加载antd图标：

```js
alias: {
    '@ant-design/icons/lib/dist$': resolve(__dirname, './plugins/antd-icons.js')
}
```

- css: 全局css

- plugins: 插件，在 根vue.js应用 实例化之前需要运行。

- modules：Nuxt.js 扩展，可以扩展它的核心功能。模块是在启动 Nuxt 时按顺序调用的函数。例如：'@nuxtjs/axios', '@nuxtjs/proxy', '@nuxtjs/dayjs'等模块。

- build：打包配置。

例如：

```js
build: {
    // 打开打包分析
    analyze: true,
    // 扩展webpack配置
    // extend(config, ctx) {
    //   
    // },
    // webpack插件配置
    plugins: [
        new CompressionPlugin({
        algorithm: 'gzip',// 压缩算法
        test: /\.js$|\.html$|\.css$|\.png$|\.gif$|\.jpe?g$/, // 匹配文件类型
        threshold: 10240, // 对超过10k的数据压缩
        deleteOriginalAssets: false // 是否删除源文件
        })
    ],
    
    // webpack loader配置
    loaders: {
        less: {
            javascriptEnabled: true,
            modifyVars: {
                'primary-color': '#52c41a'
            }
        }
    },
    // 转义配置：如果你想用 Babel 转译特定的依赖，你可以将它们添加到build.transpile。transpile 中的每个项目都可以是包名、字符串或匹配依赖项文件名的正则表达式对象。
    transpile: ['ant-design-vue'],
    // Babel 配置：为 JavaScript 和 Vue 文件自定义 Babel 配置。.babelrc默认情况下被忽略。
    babel: {
        plugins: [
        [
            'import',
            {
            libraryName: 'ant-design-vue',
            libraryDirectory: 'es',
            // 选择子目录 例如 es 表示 ant-design-vue/es/component
            // lib 表示 ant-design-vue/lib/component
            style: true
            // 默认false，即不导入样式 , 注意 ant-design-vue 使用 js 文件引入样式
            // true 表示 import  'ant-design-vue/es/component/style' 
            // 'css' 表示 import 'ant-design-vue/es/component/style/css'
            }
        ]
        ]
    },
    // 在页面中注入一些样式变量和 mixin ，需要使用相对或绝对路径。
    // 此属性已弃用。请改用[style-resources-module](https://github.com/nuxt-community/style-resources-module/)以提高性能和更好的 DX！
    styleResources: {
        less: './assets/css/index.less'
    }
}
```

### .nvmrc 配置文件（自己添加的）

**node版本配置文件，项目需要使用该文件指定的node版本安装依赖和运行项目，否则可能出现错误**

更多目录结构信息见nuxtjs文档 [nuxtjs文档](https://nuxtjs.org/docs/directory-structure/nuxt)

## 主要技术点

- 整体布局：在layouts目中defalut.vue实现整体布局
- vuex实现公用数据状态管理：1. 顶部导航菜单选中状态管理; 2. 产品数据管理
- 服务端获取数据：在vuex的store的action中通过 nuxtServerInit 在服务端获取数据
- 路由拦截：路由中间件
- 获取后端数据：`@nuxtjs/axios`模块配置添加路径前缀和开启代理；axios响应拦截器统一处理返回数据
- 跨域处理：`@nuxtjs/proxy`模块实现反向代理
