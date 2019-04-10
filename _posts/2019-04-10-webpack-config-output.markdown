---
layout: post
title: "webpack配置之Output"
date: 2019-04-10 17:45:00 +0800
categories: post
---

output是webpack打包编译的输出文件，只能有一个。

用法： output：Object

    const path = require('path');
    module.export = {
        output: {
            filename: '[name].bundle.[chunkhash:8].js',
            chunkFilename: '[name].chunk.[chunkhash:8].js',
            path: path.join(__dirname, '../dist'),
            publciPath: '/dist',
        }
    }
    
* filename是编译生成的文件。[name]是chunk的name，[chunkhash]是chunk的hash。

* path是编译生成的文件存放的目录，是一个绝对路径；

* chunkFilename用来打包require.ensure方法中引入的模块,如果该方法中没有引入任何模块则不会生成任何chunk块文件。

    比如在main.js文件中,require.ensure([],function(require){alert(11);}),这样不会打包块文件，只有这样才会打包生成块文件require.ensure([],function(require){alert(11);require('./greeter')})，或者这样require.ensure(['./greeter'],function(require){alert(11);})。
    
    chunk的hash值只有在require.ensure中引入的模块发生变化,hash值才会改变。
    
    注意:对于不是在ensure方法中引入的模块,此属性不会生效,只能用CommonsChunkPlugin插件来提取。

* publicPath是一个公共地址，用于处理静态资源的引用地址问题，比如图片的地址路径问题。尤其是在你打包图片生成的路径与html的不在同一个目录时，这个时候就必须用publicPath来指定图频引用径。


    {
        test: /\.(jpe?g|png|gif)$/i,
        loaders: [{
            loader: 'file-loader',
            options: {
                name: 'images/[name].[hash:7].[ext]'
            }
        }]
    },
    
对于文件中引入的图片，webpack打包后，都会在dist/images/下生成图片文件，如果不配置publicPath，那引用就会出错（路径不对）。

public既可以是相对路径也可以是绝对路径。绝对路径一般就是放置静态资源的CDN。

问：css引入的图片打包后为什么找不到资源？

第一种方法就是以上方法，路径注意用相对路径

第二种方法是改变ExtractTextPlugin的css路径publicPath，原因是因为css引入图片再打包后，style-loader无法设置自己的publicPath。

    {
        test: /\.less/,
        use: ExtractTextPlugin.extract({
            use: ['css-loader?minimize', 'less-loader'],
            publicPath: '../../',
            fallback: 'style-loader'
        })
    },