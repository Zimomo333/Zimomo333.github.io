---
title: 从零搭建Vue-Router-Webpack模块化开发环境
date: 2020-08-21 20:11:40
tags: [Vue,webpack]
categories: webpack
cover: /img/vue-webpack.jpg
---

# Vue模块化开发

### 项目目的

不使用vue-cli脚手架，从头搭建模块化开发环境

`Github` 项目地址：https://github.com/Zimomo333/Vue-Router-Webpack



### 项目模块

1. Vue
2. Vue Router
3. Element-UI
4. webpack



### 安装项目依赖

```shell
npm init
npm i vue -S

npm i element-ui -S

npm i vue-router -S

npm i webpack webpack-dev-server -D			//server用于运行打包后的dist资源

<!-- webpack loader -->
npm i vue-loader vue-template-compiler -D		//解析vue文件、模板
npm i css-loader style-loader -D			//解析Element-UI的CSS文件
npm i file-loader -D					//解析Element-UI的字体文件

<!-- webpack plugin -->
npm i html-webpack-plugin -D				//自动生成注入js的index.html
```



### 目录结构

![](directory.PNG)

结构说明：

| 文件/目录名         | 作用                                        |
| ------------------- | ------------------------------------------- |
| `webpack.config.js` | `webpack` 配置文件                          |
| `routes.js`         | `Vue Router` 配置文件                       |
| `main.js`           | 全局配置，`webpack`打包入口                 |
| `App.vue`           | `Vue` 根组件                                |
| `public/index.html` | `HtmlWebpackPlugin` 自定义`index.html` 模板 |
| `views`目录         | 页面组件（业务页面）                        |
| `components`目录    | 公用组件（导航栏、面包屑）                  |
| `dist`目录          | `webpack`打包输出目录                       |
| `node_modules`目录  | `npm` 模块下载目录                          |



### main.js

```javascript
import App from './App.vue'
import Vue from 'vue'
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'	//ElementUI样式
import router from './router'

Vue.use(ElementUI);

new Vue({
    el: '#app',
    router,
    render: h => h(App)
})
```



### router.js

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

export const routes = [
    { path: '/info', name: '个人中心', icon: 'el-icon-user-solid', component: () => import('../views/info.vue') },
    { path: '/orders', name: '我的订单', icon: 'el-icon-s-order', component: () => import('../views/orders.vue') }
]

const router = new VueRouter({
    routes // (缩写) 相当于 routes: routes
})

export default router
```



### webpack.config.js

```javascript
const path = require('path')
const VueLoaderPlugin = require('vue-loader/lib/plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/main.js',
    output: {
        path: path.resolve(__dirname, './dist'),
        filename: 'bundle.js'               // 打包输出的js包名
    },
    devServer: {
        contentBase: './dist'               // server运行的资源目录
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            {
                test: /\.css$/,             // 处理ElementUI样式
                loader: "style-loader!css-loader"
            },
            {
                test: /\.(woff|ttf)$/,	    // 处理ElementUI字体
                loader: 'file-loader'
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin(),              // vue-loader伴生插件，必须有！！！
        new HtmlWebpackPlugin({             // 自动生成注入js的index主页
            title: 'Vue-wepack demo',
            template: './public/index.html' // 自定义index模板
        })
    ]
}
```





### `public/index.html`模板

默认生成的`index.html `没有 id="app" 挂载点，必须使用自定义模板

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title></title>
    </head>
    <body>
        <div id="app">		<!-- App.vue根组件 挂载点 -->
        </div>
    </body>
</html>
```



### App.vue

`import { xxx } from './xxx' ` 指定需要引用的模块

`<style scoped>`  scoped 范围css 防止全局污染

```vue
<template>
    <div id="app">
        <el-row class="tac">
            <el-col :span="4">
                <h3>导航栏</h3>
                <el-menu
                :default-active="this.$route.path"
                class="el-menu-vertical-demo"
                router
                >
                <el-menu-item v-for="item in routes" :key="item.path" :index="item.path">
                    <i :class="item.icon"></i>
                    <span slot="title">{{item.name}}</span>
                </el-menu-item>
                </el-menu>
            </el-col>
            <el-col :span="16">
                <h3>正文</h3>
                <router-view />
            </el-col>
        </el-row>
    </div>
</template>

<script>
import { routes } from './router'   // {}指定需要引用的模块
export default {
    name: 'App',
    data(){
        return {
            routes
        }
    }
}
</script>

<style scoped>  /* scoped 范围css 防止全局污染 */
h3 {
    text-align: center;
}
</style>
```



### 项目展示

![](display.gif)