---
layout: post
title: "webpack配置之Loader"
date: 2019-04-10 17:47:00 +0800
categories: post
---

Loader是对应用程序中资源的转换。它们是（运行在    Node.js中的）函数，可以将资源文件作为参数的来源，然后返回新的资源文件。

#### 用法

rules: Array(Object)

配置一项rule大概有： 

* 条件匹配： 痛test、include、exclude三个配置来命中Loader要应用的规则文件。
* 应用规则：对选中的文件通过use配置项来应用loader，可以应用一个loader，也可以应用多个loader（应用顺序从右到左），同时还可以给loader传参。
* 重置顺序：一组loader的应用顺序是从右到左，通过exforce选项可以让其中一组loader的执行顺序放在最前或者最后。

例子：

安装相对应loader
    
    npm install --save-dev babel-loader
    npm install --save-dev css-loader sass-loader
    
    //webpack.config.js
    module: {
        rules: [
            {
                test: /\.js$/, 
                use: ['babel-loader?cacheDirectory'],
                include: path.resolve(__dirname, 'src')
            },
            {
                test: /\.scss$/,
                use: ['style-loader', 'css-loader', 'sass-loader'],
                exclude: path.resolve(__dirname, 'node_modules')
            }
        ]
    }

Loader的配置除了以上用法，还有其他种用法。

注意，根据配置选项，下面的规范定义了同等的 loader 用法：

    //一组loader
    {
        test: /\.css$/, 
        [loader]: 'css-loader'
    }
    // or equivalently
    {
        test: /\.css$/, 
        [use]: 'css-loader'
    }
    // or equivalently
    {
        test: /\.css$/, 
        [use]: {
            loader: 'css-loader',
            options: {}
        }
    }
    
    //多组loader
    {
        test: /\.css$/, 
        [loaders/use]: [
            {
                [loader]: 'css-loader',
                options: {},
            }
        ]
    }
    
#### Lodaer特性

* loader支持链式传递。能够对资源使用流水线。loader链式地按照先后顺序进行编译。loader链中的第一个loader返回值给下一个loader。在最后一个loader，返回所预期的Javascript。
* loader可以是同步或者异步函数。
* loader运行在Node.js中，并且能够执行任何可能的操作。
* loader接受查询参数，用于loader间传递配置。
* loader也可以使用options对象进行配置。
* 除了使用 package.json 常见的 main 属性，还可以将普通的 npm 模块导出为 loader，做法是在 package.json 里定义一个 loader 字段。
* 插件(plugin)可以为 loader 带来更多特性。
* loader 能够产生额外的任意文件。
