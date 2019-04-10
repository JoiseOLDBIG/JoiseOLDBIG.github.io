---
layout: post
title: "webpack配置之Entry入口"
date: 2019-04-10 17:44:00 +0800
categories: post
---

entry是webpack配置模块的入口。webpack进行构建的第一步将从入口开始搜寻及地柜解析出所有入口依赖模块。

#### 单个入口一个chunk

用法：entry: String

    module.export = {
        entry: './src/main.js'
    }


#### 多个入口一个chunk

用法：entry: Array<String>

    module.export = {
        entry: ['./src/entry1.js', './src/entry2.js']
    }


#### 多个入口多个chunk

用法：entry: {[entryChunkName: string]: String|Array<String>}

    moudle.export = {
        entry: {
            a: './src/main.js',
            b: ['./src/entry1.js', './src/entry2.js']
        }
    }
    
配置多个入口，每个入口生成一个chunk

常用场景：

    moudle.export = {
        entry: {
            main: './src/app.js',
            vendor: 'iview'
        }
    }
    
这样配置，webpack打包会出来两个文件。

iview是公共库，单独打包，这样就可以利用缓存来加快页面载入速度。

同时必须配合CommonsChunkPlugin插件，不然iview会同时出现在main bundle和 vendor bundle中。

    var webpack = require('webpack');
    var path = require('path');
    
    module.exports = function(env) {
        return {
            entry: {
                main: './src/app.js',
                vendor: 'iview'
            },
            output: {
                filename: '[chunkhash].[name].js',
                path: path.resolve(__dirname, 'dist')
            },
            plugins: [
                new webpack.optimize.CommonsChunkPlugin({
                    name: 'vendor' // 指定公共 bundle 的名字。
                })
            ]
        }
    }
    

但是，如果我们改变应用的代码并且再次运行 webpack，可以看到 vendor 文件的 hash 改变了。即使我们把 vendor 和 main 的 bundle 分开了，也会发现 vendor bundle 会随着应用代码改变。

这意味着我们任然无法从浏览器缓存机制中受益，因为 vendor 的 hash 在每次构建中都会改变，浏览器也必须重新加载文件。

这里的问题在于，每次构建时，webpack 生成了一些 webpack runtime 代码，用来帮助 webpack 完成其工作。当只有一个 bundle 的时候，runtime 代码驻留在其中。但是当生成多个 bundle 的时候，运行时代码被提取到了公共模块中，在这里就是 vendor 文件。

为了防止这种情况，我们需要将运行时代码提取到一个单独的 manifest 文件中。尽管我们又创建了另一个 bundle，其开销也被我们在 vendor 文件的长期缓存中获得的好处所抵消。

    var webpack = require('webpack');
    var path = require('path');
    
    module.exports = function(env) {
        return {
            entry: {
                main: './src/app.js',
                vendor: 'iview'
            },
            output: {
                filename: '[chunkhash].[name].js',
                path: path.resolve(__dirname, 'dist')
            },
            plugins: [
                new webpack.optimize.CommonsChunkPlugin({
                    names: ['vendor', 'manifest'] // 指定公共 bundle 的名字。
                })
            ]
        }
    };
    
webpack打包完成之后将会产生三个文件main、vendor、manifest