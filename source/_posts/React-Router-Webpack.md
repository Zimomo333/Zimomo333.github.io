---
title: 从零搭建React-Router-Webpack模块化开发环境（一）
date: 2020-09-11 20:11:40
tags: [React,React-Router,webpack]
categories: React
cover: /img/react-webpack.jpg
---

# React-Router-Webpack 模块化开发

### 项目目的

不使用create-react-app，从头搭建 `React` 模块化开发环境

Github仓库地址：https://github.com/Zimomo333/React-Router-Webpack



### 项目功能

1. 导航栏（折叠）
2. 面包屑
3. React-Router（路由功能）
4. Wepack打包压缩，热加载本地服务器



### 项目依赖

1. React
2. React Router（react-router-dom、react-router-config）
3. Ant Design
4. webpack（@babel/preset-react，jsx转译器）

```shell
npm init
npm i react react-dom -S

npm i antd -S

npm i react-router-dom -S		// 核心库
npm i react-router-config -S	// 官方路由配置助手(类似Vue Router，集中配置式路由)

npm i webpack webpack-cli webpack-dev-server -D         // 热加载本地server，用于运行打包后的dist资源

<!-- webpack loader -->
npm i babel-loader @babel/core @babel/preset-env @babel/preset-react	// 解析jsx语法
npm i css-loader style-loader -D            // 解析 antd 的CSS文件

<!-- webpack plugin -->
npm i html-webpack-plugin -D                // 自动生成注入js的index.html
```



### 目录结构

![](directory.PNG)

结构说明：

| 文件/目录名         | 作用                                         |
| ------------------- | -------------------------------------------- |
| `webpack.config.js` | `webpack` 配置文件                           |
| `routes.js`         | `React Router Config` 集中配置式路由文件     |
| `index.js`          | `React`渲染入口，全局配置，`webpack`打包入口 |
| `App.js`            | 根组件                                       |
| `public/index.html` | `HtmlWebpackPlugin` 自定义`index.html` 模板  |
| `views`目录         | 页面组件（业务页面）                         |
| `components`目录    | 公用组件（导航栏、面包屑）                   |
| `dist`目录          | `webpack`打包输出目录                        |
| `node_modules`目录  | `npm` 模块下载目录                           |







## React-Router

### index.js

```javascript
import React from 'react'
import ReactDOM from 'react-dom'
import { HashRouter } from "react-router-dom";
import { renderRoutes } from 'react-router-config';     // 官方路由配置助手(类似Vue Router，集中式配置)
import routes from './router'

import 'antd/dist/antd.css';

ReactDOM.render(
    <HashRouter>
        {/* 路由根入口，选择在index.js放置根入口，是为了给App组件传递prop.location，用以获取当前路径构建面包屑 */}
        { renderRoutes(routes) }
    </HashRouter>,
    document.getElementById("root")
);
```



### router.js

```javascript
import { ProfileOutlined, UserOutlined } from '@ant-design/icons';
import React, { PureComponent } from 'react'
import App from './App'
import Info from './views/Info'
import Orders from './views/orders/index'
import MyOrders from './views/orders/myOrders'
import Submit from './views/orders/submit'

const routes = [
    {
        component: App,
        routes: [
            {
                path: '/info',
                component: Info,
                title: '个人中心',
                icon: UserOutlined
            },
            { 
                path: '/orders',
                component: Orders,
                title: '订单管理',
                icon: ProfileOutlined,
                routes: [
                    {
                        path: '/orders/my-orders',
                        component: MyOrders,
                        title: '我的订单',
                        icon: ProfileOutlined
                    },
                    {
                        path: '/orders/submit',
                        component: Submit,
                        title: '提交订单',
                        icon: ProfileOutlined
                    }
                ]
            }
        ]
    }
]

export default routes
```







## Webpack

### webpack.config.js

```javascript
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, './dist'),
        filename: 'bundle.js'               // 打包输出的js包名
    },
    //mode: "production",
    devServer: {
        contentBase: './dist'               // server运行的资源目录
    },
    module: {
        rules: [
            {
                test: /\.jsx?$/, // jsx/js文件的正则
                exclude: /node_modules/, // 排除 node_modules 文件夹
                use: {
                    loader: 'babel-loader',
                    options: {
                        // babel 转义的配置选项
                        presets: [
                            "@babel/preset-react",      // 转义JSX
                            // "@babel/preset-env"      // 转移ES6+
                        ],
                        plugins: [
                            "@babel/plugin-proposal-class-properties"  // 解决报错 Support for the experimental syntax 'classProperties' isn't currently enabled
                        ]
                    }
                }
            },
            {
                test: /\.css$/,             // 处理 antd 样式
                loader: "style-loader!css-loader"
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({             // 自动生成注入js的index主页
            template: './public/index.html' // 自定义index模板
        })
    ]
}
```



### `public/index.html`模板

默认生成的`index.html `没有 id="root" 挂载点，必须使用自定义模板

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>react-router-wepack demo</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```







## 组件

### App.js

从父路由传来的prop.location中获取当前路径

matchRoutes方法 根据当前路径，获取 路由匹配历史 数组，用以构建面包屑

```react
import React from 'react'
import ReactDOM from 'react-dom'

import { matchRoutes, renderRoutes } from 'react-router-config';
import { Link } from "react-router-dom";
import routes from './router'

import { MenuUnfoldOutlined, MenuFoldOutlined } from '@ant-design/icons';
import Sidebar from './components/Sidebar'
import { Breadcrumb } from 'antd';
import './App.css'

import { Layout } from 'antd';
const { Header, Sider, Content } = Layout;

export default class App extends React.Component {
    state = {
        collapsed: false
    };
    
    toggle = () => {
        this.setState({
            collapsed: !this.state.collapsed
        });
    };

    render() {
        // 从父路由传来的prop.location中获取当前路径
        // matchRoutes方法 根据当前路径，获取 路由匹配历史 数组
        const history = matchRoutes(routes, this.props.location.pathname).slice(1);     //slice 去除首页 '/' 路由历史
        return (
            <div>
                <Layout>
                    <Sider trigger={null} collapsible collapsed={this.state.collapsed}>
                        <div className="logo" >React-Router-Webpack</div>
                        <Sidebar />
                    </Sider>
                    <Layout className="site-layout">
                        <Header className="site-layout-background" style={{ padding: 0 }}>
                            {React.createElement(this.state.collapsed ? MenuUnfoldOutlined : MenuFoldOutlined, {
                                className: 'trigger',
                                onClick: this.toggle,
                            })}
                            <Breadcrumb style={{ display: "inline" }}>
                                <Breadcrumb.Item><Link to="/">首页</Link></Breadcrumb.Item>
                                {
                                    history.map( (item,index) => 
                                            <Breadcrumb.Item key={index} >
                                                <Link to={item.route.path}>{item.route.title}</Link>
                                            </Breadcrumb.Item>
                                    )
                                }
                            </Breadcrumb>
                        </Header>
                        <Content
                            className="site-layout-background"
                            style={{
                                margin: '24px 16px',
                                padding: 24,
                                minHeight: 700
                            }}
                        >
                            {/* child routes won't render without this */}
                            {/* 根据父组件传来的路由信息，渲染当前路由下的子路由所对应的组件 */}
                            { renderRoutes(this.props.route.routes) }
                        </Content>
                    </Layout>
                </Layout>
            </div>
        )
    }
}
```



### Sidebar.js(导航栏组件)

```react
import React from 'react'
import ReactDOM from 'react-dom'
import { Link } from "react-router-dom";
import { Menu } from 'antd';
import  routes from "../router"

const { SubMenu } = Menu;

class Sidebar extends React.Component {

    nested(routes) {    //递归渲染嵌套导航栏
        return (
            routes.map( route => {
                if(!route.routes){
                    return (
                        <Menu.Item key={route.path} icon={<route.icon />}>
                            <Link to={route.path}>{route.title}</Link>
                        </Menu.Item>
                    );
                } else {
                    return (
                        <SubMenu key={route.path} icon={<route.icon />} title={route.title}>
                            { this.nested(route.routes) }
                        </SubMenu>
                    );
                }
            })
        );
    }

    render() {
        return (
            <div>
                <Menu
                    onClick={this.handleClick}
                    defaultSelectedKeys={['1']}
                    defaultOpenKeys={['sub1']}
                    mode="inline"
                    theme="dark"
                >
                {/* 取route[0] 导航栏省略外层App组件 */}
                { this.nested(routes[0].routes) }
                </Menu>
            </div>
        );
    }
}

export default Sidebar
```



### 项目展示

![](display.gif)