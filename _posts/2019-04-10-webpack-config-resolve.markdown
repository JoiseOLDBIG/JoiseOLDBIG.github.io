---
layout: post
title: "webpack配置之Resolve"
date: 2019-04-10 17:52:00 +0800
categories: post
---

webpack在启动后会从配置的入口模块触发找出所有依赖的模块，Resolve配置webpack如何寻找模块对应的文件。webpack内置JavaScript模块化语法解析功能，默认会采用模块化标准里约定好的规则去寻找，但你可以根据自己的需要修改默认的规则。

#### 用法

resolve: Object

例子： 

    resolve: {
        extensions: ['*', '.js', '.vue', '.css'],
        alias: {
            componets: './src/components/'
         }
    },
    
##### alias：把导入语句中的compontes关键字替换成./src/components/。

当你通过import Button from 'components/button'导入时，实际上被alias等价替换成import Button from './src/components/button'。

##### extensions: 在导入语句没带文件后缀时，webpack会自动带上后缀去尝试访问文件是否存在。resolve.extensions用于配置在尝试过程中用到的后缀列表.

##### modules:配置webpack去哪些目录下寻找第三方模块。默认是去node_modules目录下寻找。有时你的项目中会有一些模块大量被其他模块依赖和导入，由于其他模块的位置分布不定，针对不同的文件都要去计算被导入模块文件的相对路径，这个路径有时候会很长。

例如：import './../../components/button'，这时你可以利用modules配置项优化，假如那些大量导入的模块都在./src/components目录下：

    modules:['./src/components', 'node_modules']