---
layout: post
title: "webpack配置之Plugin"
date: 2019-04-10 17:48:00 +0800
categories: post
---

插件plugin能弥补loader不能实现的功能。

webpack 插件是一个具有 apply 属性的 JavaScript 对象。 apply 属性会被 webpack compiler 调用，并且 compiler 对象可在整个 compilation 生命周期访问。

    //ConsoleLogOnBuildWebpackPlugin.js
    
    function ConsoleLogOnBuildWebpackPlugin() {
    
    };
    
    ConsoleLogOnBuildWebpackPlugin.prototype.apply = function(compiler) {
      compiler.plugin('run', function(compiler, callback) {
        console.log("webpack 构建过程开始！！！");
    
        callback();
      });
    };


#### 用法

plugins: Array(new实例)

例子：

    //webpack.config.js
    plugins: [
        new ExtractTextPlugin({
            filename: '[name].[chunkhash:8].css',
            allChunks: true
        }),
        new HtmlWebpackPlugin({
            template: './index_template.html',
            filename: './index.html',
            favicon: path.resolve('src/img/favicon.ico'),
            showErrors: true,
            inject: 'body',
            complile: true,
            chunks: 'all',
        })
    ]