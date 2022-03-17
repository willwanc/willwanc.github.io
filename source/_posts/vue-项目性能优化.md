---
title: vue 项目性能优化
tags: ["性能优化"]
categories: ["vue"]
date: 2022-02-15 09:47:22
---

# vue 项目性能优化

性能优化可以从三个方面来进行，一是代码层面的优化，二是项目打包的优化，三是项目部署的优化。

## 代码层面的优化

### 1. 合理使用 v-if 和 v-show

v-if 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

v-if 也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

相比之下，v-show 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

一般来说，v-if 有更高的切换开销，而 v-show 有更高的初始渲染开销。因此，**如果需要非常频繁地切换，则使用 v-show 较好；如果在运行时条件很少改变，则使用 v-if 较好。**

### 2. 区分 computed、watch 和 methods 的使用场景

- computed: 计算属性，是基于它们的响应式依赖进行缓存的，只在相关响应式依赖发生改变时它们才会重新求值。**如果想要使用缓存，应该使用计算属性。**
- methods: 方法，调用方法都会再次执行方法。**如果不想使用缓存，应该使用方法。**
- watch: 侦听属性，是另一种观察和响应 Vue 实例上的数据变动的方式。**当需要在数据变化时执行异步或开销较大的操作时，应当使用侦听属性。**

### 3. 避免 v-if 和 v-for 用在同一个元素上

因为当它们处于同一节点，v-for 的优先级比 v-if 更高，这意味着 v-if 将分别重复运行于每一次 v-for 循环中，这显然会影响渲染速度。**如果只想渲染列表的部分项，最好使用计算属性过滤掉不想渲染的项目。如果是有条件地跳过循环的执行，那么可以将 v-if 置于外层元素 (或 `<template>`) 上。**

**为什么 v-for 比 v-if 具有更高的优先级？**

Vue 最终是通过 render 函数来渲染页面的，在生成 render 函数过程中，v-for 先于 v-if 判断执行，所以 v-for 比 v-if 具有更高的优先级。

### 4. 给 v-for 循环项加上 key 提高 diff 计算速度

详细原理参见 diff 计算。

### 5. 利用 Object.freeze()冻结不需要响应式变化的数据

**冻结原理**：

Vue 组件初始化过程中，会把 data 传入 observe 函数中进行数据劫持，把 data 中的数据都转换成响应式的。

在 observe 函数内部调用 defineReactive 函数处理数据，配置 getter/setter 属性，转成响应式，如果使用 Object.freeze()将 data 中某些数据冻结了，也就**是将其 configurable 属性（可配置）设置为 false。**

**defineReactive 函数中检测数据上某个 key 对应的值的 configurable 属性是否为 false，若是就直接返回**，若不是继续配置 getter/setter 属性。

```js
export function defineReactive(obj, key, val, customSetter, shallow) {
  //...
  const property = Object.getOwnPropertyDescriptor(obj, key); //获取obj[key]的属性
  if (property && property.configurable === false) {
    return;
  }
  //...
}
```

### 6. 利用 v-once 处理只会渲染一次的元素或组件

如果某个组件包含了大量静态内容，可以在根元素上添加 v-once 指令以确保这些内容只计算一次然后缓存起来。**注意：和 v-if（优先级较高） 一起使用时，v-once 不生效。在 v-for 循环内的元素或组件上使用，必须加上 key**

v-once 只渲染一次元素或组件实现原理：

如果某个虚拟 DOM 节点的缓存存在，且虚拟 DOM 节点不是在 v-for 中直接返回该虚拟 DOM 节点缓存，如果该虚拟 DOM 节点没有缓存，则调用 genStatic 方法中存在 staticRenderFns 数组中的渲染函数，渲染出虚拟 DOM 节点且存在 cached ，以便下次不用重新渲染直接返回该虚拟 DOM 节点，并同时调用 markOnce 方法在该虚拟 DOM 节点上加上 isOnce 标志，值为 true。

如果有定义 v-for ，最终会调用 \_o(${genElement(el, state)},${state.onceId++},${key}) ,其中\_o 方法就是 src\core\instance\render-helpers\render-static.js 中的 markOnce 方法，其作用是在生成的虚拟 DOM 节点上加上 isOnce 标志，为 true 代表该虚拟 DOM 节点是静态节点，当 patch 时，会判断 vnode.isOnce 是否为 true，为 true 时，直接返回旧节点，不进行比对，相当实现渲染一次。

### 7.提前过滤掉后端返回的非必须数据，优化 data 选项中的数据结构

接收服务端传来的数据，常常会有一些渲染页面时用不到的数据。所以要先把服务端传来的数据中那些渲染页面用不到的数据先过滤掉。然后再赋值到 data 中。可以避免去劫持那些渲染页面不需要的数据，从而提高渲染速度。

### 8. 避免在 v-for 循环中读取 data 中数组类型的数据，在需要读取时，取出数组中要用的数据再复值给 data

原因：在组件 data 初始化的过程中，defineReactive 函数给每个数据的 getter 函数中定义了每个次读取该数据进行的依赖收集。如果是一个数组类型的数据，就遍历数组中的元素对其进行依赖收集，所以会造成多余的循环，从而影响性能。

### 9. 防抖和节流

防抖（debounce）和节流(throttle)是针对用户操作的优化。防抖可以用在 input, keyup，click 等用户可能频繁触发的事件中，节流一般用在比 input, keyup 更频繁触发的事件中，如：resize, touchmove, mousemove, scroll 等。

### 10. 图片懒加载

安装 vue-lazyload 插件。在 main.js 中引入：

```js
import VueLazyload from "vue-lazyload";
Vue.use(VueLazyload, {
  preLoad: 1.3, //预载高度比例
  error: "dist/error.png", //加载失败显示图片
  loading: "dist/loading.gif", //加载过程中显示图片
  attempt: 1, //尝试次数
});
```

在项目中使用：

```vue
<img v-lazy="/static/img/1.png">
```

### 11. 第三方依赖按需加载

第三方依赖按需加载的方法，一般相应文档都有介绍。例如：

- [ant-design-vue按需加载](https://www.antdv.com/docs/vue/getting-started-cn/#%E6%8C%89%E9%9C%80%E5%8A%A0%E8%BD%BD)



### 12. 利用挂载节点会被替换的特性优化白屏问题

白屏问题：Vue 项目有个缺点，首次渲染会有一段时间的白屏。原因是首次渲染时需要加载很多资源，如 js、css、图片。但是如果遇上网络慢的情况，首先加载是 index.html 页面，其是没有内容，就会出现白屏。

Vue 选项中的 render 函数若存在，则 Vue 构造函数不会从 template 选项或通过 el 选项指定的挂载元素中提取出的 HTML 模板编译渲染函数。也就是说渲染时，会直接用 render 渲染出来的内容替换 `<div id="app"></div>`。

```vue
import Vue from 'vue' import App from './App.vue' new Vue({ render: h => h(App)
}).$mount('#app')
```

如果 `<div id="app"></div>` 里面有内容，就不会出现白屏。所以我们可以在 `<div id="app"></div>` 里添加首屏的静态页面。等真正的首屏加载出来后就会把`<div id="app"></div>` 这块结构都替换掉，给人一种视觉上的误差，就不会产生白屏。

## 项目打包的优化

### 常用打包优化手段

主要包括：

- 路由组件懒加载和聚合打包
- 第三方依赖：分包通过CDN引入或者代码分割以利用客户端缓存
- js、css、图片等静态资源的合并压缩
- Prefetch（预获取）和Preload（预加载）模块
- 开启gzip压缩

### 1. 路由组件懒加载和聚合打包

**路由组件懒加载：**利用 import()异步引入组件实现路由组件懒加载，在访问相应的页面时再去加载对应页面的js文件。

```js
const routes = [
  { path: '/', component: () => import('./Dashboard.vue') }
  { path: '/contact', component: () => import('./Contact.vue') }
]
```

**聚合打包：**一般我们的一个页面的bundle可能非常的小，我们可以把一个模块的所有路由的页面都打包到一个bundle中来减少HTTP请求次数，在import函数加入 `/* webpackChunkName:'group-superAdmin' */` ，其中webpackChunkName的名称 group-superAdmin 是自己定义的。

```js
const router = [
  {
    path: 'superAdminAccountList',
    name: 'SuperAdminAccountList',
    component: () => import(/* webpackChunkName:'group-superAdmin' */ '@/activity/superAdmin/AccountList'),
  },
  {
    path: 'superAdminCreateAccount',
    name: 'SuperAdminCreateAccount',
    component: () => import(/* webpackChunkName:'group-superAdmin' */ '@/activity/superAdmin/CreateAccount'),
  },
  {
    path: 'superAdminRoleList',
    name: 'SuperAdminRoleList',
    component: () => import(/* webpackChunkName:'group-superAdmin' */ '@/activity/superAdmin/RoleList'),
  },
  {
    path: 'superAdminCreateRole/:id?/:look?',
    name: 'SuperAdminCreateRole',
    component: () => import(/* webpackChunkName:'group-superAdmin' */ '@/activity/superAdmin/CreateRole'),
  },
];
```


### 2. 利用externals分离出第三方依赖并用CDN引入

很多我们引入的第三份依赖文件是可以利用 CDN 上的资源的，这样可以大大加快我们访问这些资源的速度，同时减轻我们服务器的压力。但是也存在 CDN 服务器故障的风险。

具体配置方式见：https://webpack.docschina.org/configuration/externals/#externals

### 3. 代码分割（vue-cli4 项目默认已经配置）

代码分割是 webpack 中最引人注目的特性之一。此特性能够把代码分割到不同的 bundle 中。代码分割可以用于获取更小的 bundle，以及控制资源加载优先级和充分利用浏览器的缓存机制。

上面第2项通过CDN使用是第三方依赖，具有不稳定性，万一CDN突然挂了，系统也就崩了，有一定的风险。也可以用SplitChunks实现代码分割，第三方依赖还可以在自己服务器上，减少风险。


这部分 vue-cli4 项目默认已经配置。具体配置如下：

```js
optimization: {
    splitChunks: {
      cacheGroups: {
        vendors: {
          name: 'chunk-vendors',
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          chunks: 'initial'
        },
        common: {
          name: 'chunk-common',
          minChunks: 2,
          priority: -20,
          chunks: 'initial',
          reuseExistingChunk: true
        }
      }
    }
}
```

如果需要修改配置，可参考 [SplitChunksPlugin配置](https://webpack.docschina.org/plugins/split-chunks-plugin/#defaults)

### 4. 利用 MiniCssExtractPlugin 插件提取 组件内 css 到单独的文件（vue-cli4 项目默认已经配置）

MiniCssExtractPlugin：该插件将 CSS 提取到单独的文件中。它为每个包含 CSS 的 JS 文件创建一个 CSS 文件。它支持 CSS 和 SourceMaps 的按需加载。

在用 Vue Cli4 搭建的 Vue 项目中是用 css.extract 来控制 MiniCssExtractPlugin 插件是否启用，在生产环境中，css.extract 默认是开启的。

### 5. 利用 OptimizeCssnanoPlugin 插件压缩和去重 css 样式文件（vue-cli4 项目默认已经配置）

在用Vue Cli4搭建的Vue项目中默认使用OptimizeCssnanoPlugin插件来压缩和去重css样式文件。

### 6. 开启optimization.minimize来压缩js代码（vue-cli4 项目默认已经配置）

optimization.minimize 选项有两个值true 和false ，为 true开启压缩js代码，为false 关闭压缩js代码。

在生产环境中默认为 true ，在开发环境中默认为false 。

在vue-cli4 项目中默认用TerserPlugin插件来压缩js代码。
默认的配置已经很好了，如果想修改TerserPlugin插件的配置，可以参考[vue-cli修改插件配置文档](https://cli.vuejs.org/zh/guide/webpack.html#%E4%BF%AE%E6%94%B9%E6%8F%92%E4%BB%B6%E9%80%89%E9%A1%B9)和[插件文档](https://www.npmjs.com/package/terser)：

如果想用其它插件来压缩js代码，可以在 optimization.minimizer 选项中添加，其值为数组。

用chainWebpack来添加，其中WebpackPlugin为插件名称，args为插件参数。

```js
const WebpackPlugin = require(插件名称)
module.exports = {
    chainWebpack: config =>{
        config.optimization
            .minimizer(name)
            .use(WebpackPlugin, args)
    },
}
```


### 7. 利用image-webpack-loader进行压缩图片

用Vue Cli4搭建的Vue项目中，图片是没进行压缩就直接用url-loader和file-loader处理的。**可以用image-webpack-loader进行压缩图片后再给url-loader和file-loader处理。**

先安装 image-webpack-loader。**注意：如果用yarn或npm 安装可能会失败（包括修改安装源），此时可以用cnpm安装**

```bash
// 先安装cnpm
npm install cnpm -g --registry=https://registry.npm.taobao.org

// 安装image-webpack-loader
cnpm install --save-dev  image-webpack-loader
```

在vue.config.js种添加配置：

```js
module.exports = {
    chainWebpack: config =>{
        config.module
            .rule('images')
            .use('imageWebpackLoader')
            .loader('image-webpack-loader')
            .options({
                // 生产环境禁用压缩，默认启用。
                disable: process.env.NODE_ENV === 'development',
                // 控制压缩JPEG图像的配置
                mozjpeg:{
                    quality:75 // 压缩质量，范围0（最差）至100（最好）
                },
                // 控制压缩PNG图像的配置，默认启用。
                optipng:{
                    OptimizationLevel:3 // 优化级别，0-7，数值越高，压缩质量越好，但是速度越慢，默认为3。
                },
                // 控制压缩PNG图像的配置，默认启用。
                pngquant:{
                    speed:4,
                    quality:[0.2,0.5]
                },
                // 控制压缩GIF图像的配置，默认启用。
                gifsicle:{
                    OptimizationLevel:1 // 优化级别，1-3；较高的水平需要更长的时间，但可能会有更好的效果。
                },
                // 将JPG和PNG图像压缩为WEBP，默认不启用，需要配置后才启用。启用后，可以将JPG和PNG图像压缩输出大小更小的图片，但比用mozjpeg、optipng、pngquant压缩更耗时
                webp:{
                    quality:75, // 品质，0-100，默认为75，值越高品质越好。
                    lossless:true, // 是否无损压缩，默认为false
                    nearLossless:75 // nearLossless 使用额外的有损预处理步骤进行无损编码 0-100
                }
            })
    },
}
```


### 8. Prefetch（预获取）和Preload（预加载）资源（vue-cli4 项目默认已经配置）

Webpack v4.6.0+ 增加了对预获取和预加载的支持。

在声明 import 时，使用下面这些内置指令，可以让 webpack 输出 "resource hint(资源提示)"，来告知浏览器：

- prefetch(预获取)：将来某些导航下可能需要的资源
- preload(预加载)：当前导航下可能需要资源

prefetch 和 preload 的不同之处：

**preload chunk 会在父 chunk 加载时，以并行方式立即开始加载。prefetch chunk 会在父 chunk 加载结束后浏览器闲置时开始加载。**

具体配置详见：https://webpack.docschina.org/guides/code-splitting/#prefetchingpreloading-modules。

vue-cli4 项目默认使用了 preload-webpack-plugin 来自动处理模块的预加载和预获取。配置如下：

```js
plugins: [
  /* config.plugin('preload') */
    new PreloadPlugin(
      {
        rel: 'preload',
        include: 'initial', // 初始模块（同步）
        fileBlacklist: [
          /\.map$/,
          /hot-update\.js$/
        ]
      }
    ),
    /* config.plugin('prefetch') */
    new PreloadPlugin(
      {
        rel: 'prefetch',
        include: 'asyncChunks' // 异步模块
      }
    ),
]
```

上述配置意为：把初始模块配置为预加载资源，把异步模块配置为预获取资源。更多配置见：[插件文档](https://www.npmjs.com/package/@vue/preload-webpack-plugin)

### 9. 在Webpack上开启gzip压缩

首先安装 compression-webpack-plugin，**这里要注意和webpack版本的兼容性，如果是webpack4请安装compression-webpack-plugin@4.0.0**

```bash
npm i compression-webpack-plugin -D
```

然后在 vue.config.js 中添加配置：

```js
const CompressionPlugin = require('compression-webpack-plugin');
module.exports = {
    configureWebpack: config =>{
        return {
            plugins: [
                new CompressionPlugin({
                    algorithm: 'gzip',// 压缩算法
                    test: /\.js$|\.html$|.\css/, // 匹配文件类型
                    threshold: 10240, // 对超过10k的数据压缩
                    deleteOriginalAssets: false // 是否删除源文件
                })
            ],
        }
    }
}
```

执行 npm run build 命令后，打开 dist 文件，会发现多出很多后缀为 .gz 的文件，这就是用gzip压缩后的文件。

该插件的具体配置项见：https://www.npmjs.com/package/compression-webpack-plugin

### 10. 打包后的文件分析和优化

此外，项目打包后的文件大小直接影响我们访问页面的加载速度，所以我们要知道哪些打包后的文件最大，最影响性能。

用[webpack-bundle-analyzer](https://www.npmjs.com/package/webpack-bundle-analyzer)插件来生成一个报告。

用 vue-cli4 创建的项目 可以直接在 package.json 的 script 中添加如下配置：

```js
"build-report": "vue-cli-service build --report"
```

然后，在命令行中运行

```bash
npm run build-report
```

打包完成后会在dist文件中生成一个report.html文件，我们在浏览器中打开它就能看到打包后的文件大小分析报告图。通过分析报告图我们可以清晰直观的看出每个打包后文件的大小，进一步对比较大和依赖包进行分析看是否能够优化。

比如：

在我的项目中用到ant-design-vue的部分图标，我发现被全部图标都被打包进项目了，此时就可以参考这两个链接进行按需引入图标：[vue.config.js#L39-L43](https://github.com/vueComponent/pro-components/blob/v2/examples/vue.config.js#L39-L43)、[icons.js](https://github.com/vueComponent/pro-layout/blob/master/examples/src/core/antd/icons.js)

还有我们常用的moment.js，默认是包含很多语言包的，这些语言包默认也被打包尽我们的代码里了。通过在vue.config.js添加如下配置，我们可以把这些语言包从打包文件中剔除：

```js
// vue.config.js

const webpack = require('webpack')

// vue.config.js
const vueConfig = {
  configureWebpack: {
    plugins: [
      // Ignore all locale files of moment.js
      new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)
    ]
  }
}
```


## 项目部署的优化

### 1. 在Nginx上开启gzip压缩

在nginx的配置文件种添加如下配置：

```conf
http {
    # 开启|关闭（off） gzip
    gzip on; 

    # 开启后如果能找到 .gz 文件，直接返回该文件，不会启用服务端压缩。
    gzip_static on; 

    # 文件大于指定 size 才压缩，以 kb 为单位。
    gzip_min_length 1k; 

    # 压缩级别，1-9，值越大压缩比越大，但更加占用 CPU，且压缩效率越来越低。
    gzip_comp_level 5; 

    # 要压缩的文件类型。
    gzip_types application/javascript image/png image/gif image/jpeg text/css text/plain;
    
    # 请求压缩的缓冲区数量和大小，以 4k 为单位，32 为倍数。
    gzip_buffers 32 4k;
    
    gzip_http_version 1.1;

    # 是否添加响应头 Vary: Accept-Encoding 建议开启。
    gzip_vary on;
}
```

#### Nginx和Webpack开启gzip压缩的区别

webpack 开启 gzip 压缩：只是在打包的时候生成相应地gz压缩文件，如果 Nginx 没有开启 gzip，浏览器在获取相应的文件是还是获取未进行 gzip 压缩的文件。
nginx 服务器开启 gzip 压缩：会将静态资源在服务端进行压缩，压缩包传输给浏览器后，浏览器再进行解压使用，这大大提高了网络传输的效率。

**总结：建议Nginx和Webpack压缩都开启压缩，且在Nginx加上gzip_static on; 的配置，减少服务器的CPU的使用。**


### 2.静态资源配置缓存策略

主要包括强缓存和弱缓存（协商缓存），具体配置有待研究。

## 参考

- [前端面试如何说解vue项目性能优化，你确定不来看看吗？](https://www.jianshu.com/p/ef44aaa41fe8)
- [Vue项目性能优化实战总结，7个实用方法](https://zhuanlan.zhihu.com/p/421604906)
- [基于vue-cli创建的项目进行打包优化](https://zhuanlan.zhihu.com/p/395587041)
- [Vue-cli3脚手架3的webpack打包优化压缩](https://segmentfault.com/a/1190000022750932)
- [ant-design-pro-vue常见问题——编译后体积很大](https://pro.antdv.com/docs/faq#%E7%BC%96%E8%AF%91%E5%90%8E%E4%BD%93%E7%A7%AF%E5%BE%88%E5%A4%A7%EF%BC%9F)
- [Vue CLI 指南](https://cli.vuejs.org/zh/guide)
- [Webpack 中文文档](https://webpack.docschina.org/configuration/)



