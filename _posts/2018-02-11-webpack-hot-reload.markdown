---
layout: post
title:  "webpack热加载探索"
date:   2018-02-11 10:57:11 +0800
categories: post
---
想对webpack有更深层次的理解，首先就从“热加载”开始。。。

在pakage.json文件中，我们都会注意到

    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.

这样一行代码，没错“webpack-dev-server”其实就是用于热加载的。

## 一、webpack-dev-server的使用方法

1、安装

npm install webpack-dev-server -g

2、写入到依赖

npm install webpack-dev-server --save-dev

-S, --save: Package will appear in your dependencies.
-D, --save-dev: Package will appear in your devDependencies.
-O, --save-optional: Package will appear in your optionalDependencies.

3、修改index.html

<script type="text/javascript" src="bundle.js"></script>

4、运行命令

webpack-dev-server --hot --inline


## 二、webpack热加载原理探索

所谓的hot reload（热加载）是每次修改完某个js文件后，页面局部更新。

webpack热加载基本原理

基本实现原理大致这样的，构建bundle的时候，加入了一段HMR(hot module replacement) runtime的js和一段和服务沟通的js。文件修改会触发webpack重新构建，服务器通过向游览器发送更新消息，游览器通过jsonp拉取更新的模块文件，jsonp回调触发模块热替换逻辑。

热加载的基本思路就是，监听本地文件修改，然后服务器推送到客户端，执行更新。

# 来源？

webpack有两个核心概念：

Code Splitting
Everything is a module

这里就先了解下Code Splitting。顾名思义Code Splitting就是把代码拆分成不同的模块，并且是在代码需要执行的时候才加载，所谓的按需加载。

webpack对模块设计上还区分了异步模块和同步模块，构建过程中自动构建成两个不同的chunk文件，异步模块按需加载。

Code Splitting还体现在对公共以来的抽离（CommonsChunkPlugin），如果一个构建过程有多个入口文件，这些入口的公共依赖可以打包成一个chunk。

webpack通过require.ensure来定义一个分离点。require.ensure在实际执行过程中触发了一个jsonp请求，这个请求回调后返回一个对象，这个对象包括了所有异步模块id与异步模块代码。

# 实现

热加载实现主要分为几部分功能

服务器构建、推送更新消息
游览器模块更新
模块更新后页面渲染

# 构建

热加载是通过内置的HotModuleReplacementPlugin实现的，构建过程中热加载相关的逻辑都在这个插件中。这个插件主要处理两部分逻辑

注入HMR runtime逻辑
找到修改的模块，生成一个补丁js文件和更新描述json文件

HMR runtime主要定义了jsonp callback方法，这个方法会触发模块更新，并且对模块新增一个module.hot相关API，这个API可以让开发者自定义页面更新逻辑。

构建过程中需要对更新的文件打包出两个文件，这两个文件名规则定义在WebpackOptionsDefaulter
this.set("output.hotUpdateChunkFilename", "[id].[hash].hot-update.js");
this.set("output.hotUpdateMainFilename", "[hash].hot-update.json");

这两个文件一个是说明更新了什么，另外一个是更新的模块代码。这两个文件生成逻辑

compilation.plugin("additional-chunk-assets", function() {
  this.modules.forEach(function(module) {
    // 对比 md5 ，标记有修改的模块
    module.hotUpdate = records.moduleHashs[identifier] !== hash;
  });
  // 更新内容对象
  var hotUpdateMainContent = {};

  // 找到更新的 js 模块
  Object.keys(records.chunkHashs).forEach(function(chunkId) {
    // 渲染更新的 js ，并且追加到 assets
    var source = hotUpdateChunkTemplate.render(...);
    this.assets[hotUpdateChunkFilename] = source;
    hotUpdateMainContent.c.push(chunkId);
  }, this);

  var source = new RawSource(JSON.stringify(hotUpdateMainContent));
  // assets 中增加 json 文件
  this.assets[hotUpdateMainFilename] = source;
});

具体的过程是在构建chunk的过程中，定义一个插件方法additional-chunk-assets，在这个方法里面通过md5对比hash修改，找到修改的模块，如果发现有模块md5修改了，那么说明有更新，这时候通过hotUpdateChunkTemplate.render生成一份更新的js文件，也就是上面定义的output.hotUpdateChunkFileName，并且在assets中追加一份json描述文件，说明更新了哪个模块以及更新的hash。

上面的代码也可以发现webpack构建过程提供了很多丰富的接口，并且追加一个output文件是非常容易的事情，只需要在assets中push一个文件即可，找到修改的文件也很方便，首先构建前遍历所有的模块，记录所有模块文件内容hash，构建完成后，在一个个对比一遍，就可以找到更新的模块。

# 服务器推送

文件更新后，首先需要打包好心的补丁文件，还需要告诉游览器文件修改了，可以拉代码了。

这一部分webpack自带了一个dev-server。当开启热加载的时候，webpack-dev-server会响应客户端发起的EventStream请求，然后保持请求不断开。这样服务器就可留意在有更新的时候直接把结果push到游览器。

服务器推送部分比较简单，构建一个node的Sever-Sent Events服务器只需要几行代码，这里有一个例子。

一次完成的构建流程大概是这样的

![png1]

上述步骤完成，热加载前两步就ok了。每次文件修改，游览器模块代码也更新了。但是就这样而言，模块更新还算不上完整的热加载，因为模块更新了，页面还没更新。前面提到构建过程中会在入口文件中加入一段HRM runtime代码，其中就有加上module.hot相关API。这个API就是提供给开发者自定义页面更新用的。

下面，我们进入热加载最后一步，页面局部更新。

runtime（运行时系统），是一套基于C语言API，包含在<objc/runtime.h>和<objc/message.h>中，运行时系统的功能是在运行期间（而不是编译期或其他时机）通过代码去动态的操作类（获取类的内部信息和动态操作类的成员），如创建一个新类、为某个类添加一个新的方法或者为某个类添加添加实例变量、属性，或者交换两个方法的实现、获取类的属性列表、方法列表等和Java中的发射技术类似。


# css hot loader

js热加载基本上是通过自动更新组件，重新渲染页面两个步骤完成了。还有一个比较重要的是css热加载，webpack官方提供的方案是style-loader。

一般的对css处理都是通过extract-text-webpack-plugin插件把css抽离到单独css文件中。但extract-text-webpack-plugin是不支持热加载的，所以css热加载需要两个步骤：

 开发环境关闭extract-text-webpack-plugin
 开启sytle-loader插件

 style-loader实际上就是通过js创建一个style标签，然后注入内联的css。因为css是内联，并且通过js注入，那么页面刷新的时候一开始是没有任何css的，这种体验会非常差，闪一下然后页面重新渲染成功。

 webpack中扩展功能有两种方式

 loader
 plugin

 一个loader是对模块进行处理，比如css处理过程可以用这样来描述

    style-loader!css-loader!less

对每个css模块会依次执行less、css-loader、style-loader处理，每个loader处理后的结果作为下一个loader的输入字符串。

插件处理的是chunk，loader处理完成后，可以得到一个依赖树，每个模块都有一个处理结果的描述。在插件里面可以对整个entry输出的内容进行一些处理，比如热加载过程中增加HRM runtime脚本，对所有的css抽离到单独的静态资源中。

css-hot-loader所做的是在css模块中注入一段脚本，所以是一个loader，并且是第一个loader，这样可以保证代码不会被extract-text-webpack-plugin抽出来。

# 总结





参考http://shepherdwind.com/2017/02/07/webpack-hmr-principle/


按需加载的原理？？？？



[png1]: {{ site.baseurl }}/assets/images/20180211-1.png