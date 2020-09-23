---
title: 从零搭建Vuex-Router-Webpack模块化开发环境（二）
date: 2020-08-23 20:11:40
tags: [Vue,Vue-Router,Vuex,webpack]
categories: Vue
cover: /img/vue-webpack.jpg
---

# Vuex-Router-Webpack 模块化开发

### 项目目的

不使用vue-cli脚手架，从头搭建模块化开发环境

Github仓库地址：https://github.com/Zimomo333/Vuex-Router-Webpack



### 项目功能

1. 导航栏（折叠）
2. 面包屑
3. Vue-Router（路由、拦截功能）
4. Vuex（中心化登录权限）
5. token存入Cookie，用户信息存入localStorage
6. Wepack打包压缩，热加载本地服务器



### 项目依赖

1. Vue
2. Vue Router
3. Vuex
4. axios异步请求、js-cookie操作Cookies
5. Element-UI
6. webpack

```shell
npm init
npm i vue -S

npm i element-ui -S

npm i vue-router -S

npm i vuex -S

npm i axios -S
npm i js-cookie -S

npm i webpack webpack-dev-server -D         // 热加载本地server，用于运行打包后的dist资源

<!-- webpack loader -->
npm i vue-loader vue-template-compiler -D   // 解析vue文件、模板
npm i css-loader style-loader -D            // 解析Element-UI的CSS文件
npm i file-loader -D                        // 解析Element-UI的字体文件

<!-- webpack plugin -->
npm i html-webpack-plugin -D                // 自动生成注入js的index.html
```



### 目录结构

![](directory.PNG)

结构说明：

| 文件/目录名         | 作用                                        |
| ------------------- | ------------------------------------------- |
| `webpack.config.js` | `webpack` 配置文件                          |
| `routes.js`         | `Vue Router` 配置文件                       |
| `store.js`          | `Vuex` 配置文件                             |
| `main.js`           | 全局配置，`webpack`打包入口                 |
| `request.js`           | 二次封装axios，错误码处理，header设置token |
| `auth.js`           | Cookies、localStorage 操作 |
| `App.vue`           | `Vue` 根组件                                |
| `public/index.html` | `HtmlWebpackPlugin` 自定义`index.html` 模板 |
| `views`目录         | 页面组件（业务页面）                        |
| `components`目录    | 公用组件（导航栏、面包屑）                  |
| `api`目录  | 请求接口目录                                |
| `dist`目录          | `webpack`打包输出目录                       |
| `node_modules`目录  | `npm` 模块下载目录                          |







## Vue-Router

### router.js

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'
import store from './store'
import { getToken } from './utils/auth'

Vue.use(VueRouter)

export const routes = [
  {
    path: '//',     // 转义 / ，防止自动省略为空
    component: () => import('./views/home.vue'),
    meta: { 
      title: '首页',
      icon: 'el-icon-s-order'
    },
    hidden: true,
    children: [
      { 
        path: '/info',
        component: () => import('./views/info.vue'),
        meta: { 
            title: '个人中心',
            icon: 'el-icon-user-solid'
        }
      },
      {
        path: '/orders',        // 以 / 开头的嵌套路径会被当作根路径
        component: () => import('./views/orders/index.vue'),    // 可写成{render: (e) => e("router-view")}，避免新建空router-view文件
        meta: { 
            title: '订单管理',
            icon: 'el-icon-s-order'
        },
        children: [
          {
            path: 'my-orders',      // 子路由不要加 /
            component: () => import('./views/orders/myOrders.vue'),
            meta: { 
                title: '我的订单',
                icon: 'el-icon-s-order'
            }
          },
          {
            path: 'submit',
            component: () => import('./views/orders/submit.vue'),
            meta: { 
                title: '提交订单',
                icon: 'el-icon-s-order'
            }
          }
        ]
      },
    ]
  },
  {
    path: '/login',
    component: () => import('./views/login.vue'),
    hidden: true
  }
]

const router = new VueRouter({
    routes // (缩写) 相当于 routes: routes
})

//路由全局前置守卫，验证token
router.beforeEach(async(to, from, next) => {
  
    // determine whether the user has logged in
    const hasToken = getToken()
  
    if (hasToken) {
      if (to.path === '/login') {
        // if is logged in, redirect to the home page
        next('/')
      } else {
        const hasGetUserInfo = store.getters.name
        if (hasGetUserInfo) {
          next()
        } else {
          try {
            // get user info
            await store.dispatch('getInfo')
  
            next()
          } catch (error) {
            await store.dispatch('resetToken')
            next('/login')
          }
        }
      }
    } else {
        /* has no token*/
        if (to.path === '/login') {   //next()才能跳出循环
            next()
        } else {
            next('/login')
        }
    }
})

export default router
```







## Vuex

### store.js

```js
import Vue from 'vue'
import Vuex from 'vuex'
import { login, logout, getInfo } from './api/user'
import { getToken, setToken, removeToken, setUserInfo, removeUserInfo } from './utils/auth'

Vue.use(Vuex)

const store = new Vuex.Store({
    state: {
        token: getToken(),
        name: '',
        avatar: ''
    },
    getters: {
        token: state => state.token,
        avatar: state => state.avatar,
        name: state => state.name
    },
    mutations: {
        RESET_STATE: (state) => {
            state.token = ''
            state.name = ''
            state.avatar = ''
        },
        SET_TOKEN: (state, token) => {
            state.token = token
        },
        SET_USERINFO: (state, userInfo) => {
            state.name = userInfo.name
            state.avatar = userInfo.avatar
        },
    },
    actions: {
        login({ commit }, loginForm) {
            const { username, password } = loginForm
            return new Promise((resolve, reject) => {
                login({ username: username.trim(), password: password }).then(response => {
                    const { data } = response
                    commit('SET_TOKEN', data.token)
                    setToken(data.token)
                    resolve()
                }).catch(error => {
                    reject(error)
                })
            })
        },
        getInfo({ commit, state }) {
            return new Promise((resolve, reject) => {
                getInfo(state.token).then(response => {
                    const { data } = response

                    if (!data) {
                        reject('Verification failed, please Login again.')
                    }

                    const { userInfo } = data
                    commit('SET_USERINFO', userInfo)
                    setUserInfo(userInfo)
                    resolve(data)
                }).catch(error => {
                    reject(error)
                })
            })
        },
        logout({ commit, state }) {
            return new Promise((resolve, reject) => {
                logout(state.token).then(() => {
                    removeToken()
                    removeUserInfo()
                    commit('RESET_STATE')
                    resolve()
                }).catch(error => {
                    reject(error)
                })
            })
        },
        // remove token
        resetToken({ commit }) {
            return new Promise(resolve => {
                removeToken() 
                commit('RESET_STATE')
                resolve()
            })
        }
    }
})

export default store
```







## Webpack

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
            title: 'Vuex-Router-Webpack demo',
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





## 组件

### SidebarItem.vue(导航栏组件)

组件自调用实现嵌套导航栏

传递basePath记录父路由路径，与子路由拼接成完整路径

```vue
<template>
  <!-- 隐藏不需要显示的路由 -->
  <div v-if="!item.hidden">
    <!-- 如果没有子路由 -->
    <template v-if="!item.children">
        <el-menu-item :key="item.path" :index="resolvePath(item.path)">
          <i :class="item.meta.icon"></i>
          <span slot="title">{{item.meta.title}}</span>
        </el-menu-item>
    </template>
    <!-- 如果有子路由，渲染子菜单 -->
    <el-submenu v-else :index="resolvePath(item.path)">
      <template slot="title">
          <i :class="item.meta.icon"></i>
          <span slot="title">{{item.meta.title}}</span>
      </template>
      <sidebar-item v-for="child in item.children" :key="child.path" :item="child" :basePath="resolvePath(item.path)" />
    </el-submenu>
  </div>
</template>

<script>
import path from 'path'

export default {
  name: 'SidebarItem',    //组件自调用，必须有name属性
  props: {                //接收父组件传参
    item: {
      type: Object,
      required: true
    },
    basePath: {     //从父组件一直拼接下来的基路径
      type: String,
      default: ''
    }
  },
  methods: {
    //拼接父子路径
    resolvePath(routePath) {
      return path.resolve(this.basePath,routePath)
    }
  }
}
</script>
```





### 项目展示

![](display.gif)