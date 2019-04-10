---
layout: post
title: "webpack配置文件"
date: 2019-04-10 17:41:00 +0800
categories: post
---

webpack是JavaScript应用程序的模块打包器。

主要有四个核心概念：

#### 入口（Entry）

entry是webpack的打包入口文件。

简单的例子如下：

    module.export = {
        entry: './src/main.js'
    }
    
#### 出口（Output）

output是webpack打包完成之后输出的文件，主要包括文件路径和文件名称。

例子如下：

    module.export = {
        output: {
            path: path.resolve(__dirname, 'dist'),
            filename: 'bundle.js'
        }
    }
    
#### 加载器（Loader）

各种加载器，可以把项目中webpack不能理解的文件（.css, .html, .scss, .jeg etc）转化成webpack能理解的模块。

例子如下：

    module.export = {
        module: {
            rules: [
                {
                    test: '/\.(js|jsx)$/',
                    use: 'babel-loader',
                }
            ]
        }
    }
    
#### 插件（Plugins）

plugins能做一些loader做不了的事情。

loader 仅在每个文件的基础上执行转换，而 插件(plugins) 最常用于（但不限于）在打包模块的“compilation”和“chunk”生命周期执行操作和自定义功能。webpack 的插件系统极其强大和可定制化。

例子如下：

    const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
    module.export = {
        plugins: [
            new HtmlWebpckPlugin({
                template: './index_template.html'
            })
        ]
    }
    
##### webpack配置文件除了这四个之外，还有resolve

webpack在启动后会从配置的入口模块出发找出所有依赖(import/required)的模块，Resolve配置webpack如何寻找模块对应的文件。

webpack内置JavaScript模块化语法解析功能，默认会采用模块化标准里约定好的规则去寻找，但你可以根据自己的需要修改默认的规则。

例子如下：

    module.export = {
        resolve: {
            alias: {
                compontets: './src/components/'
            },
            extensions: ['.js', '.json']
        }
    }
    
    