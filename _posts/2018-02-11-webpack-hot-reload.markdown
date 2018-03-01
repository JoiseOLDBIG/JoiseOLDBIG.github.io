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

### webpack热加载基本原理

基本实现原理大致这样的，构建bundle的时候，加入了一段HMR(hot module replacement) runtime的js和一段和服务沟通的js。文件修改会触发webpack重新构建，服务器通过向游览器发送更新消息，游览器通过jsonp拉取更新的模块文件，jsonp回调触发模块热替换逻辑。

热加载的基本思路就是，监听本地文件修改，然后服务器推送到客户端，执行更新。

### 来源？

#### webpack有两个核心概念：

* Code Splitting

* Everything is a module

这里就先了解下Code Splitting。顾名思义Code Splitting就是把代码拆分成不同的模块，并且是在代码需要执行的时候才加载，所谓的按需加载。

webpack对模块设计上还区分了异步模块和同步模块，构建过程中自动构建成两个不同的chunk文件，异步模块按需加载。

Code Splitting还体现在对公共依赖的抽离（CommonsChunkPlugin），如果一个构建过程有多个入口文件，这些入口的公共依赖可以打包成一个chunk。

webpack通过require.ensure来定义一个分离点。require.ensure在实际执行过程中触发了一个jsonp请求，这个请求回调后返回一个对象，这个对象包括了所有异步模块id与异步模块代码。

### 实现

热加载实现主要分为几部分功能

* 服务器构建、推送更新消息

* 游览器模块更新

* 模块更新后页面渲染


### 构建

热加载是通过内置的HotModuleReplacementPlugin实现的，构建过程中热加载相关的逻辑都在这个插件中。这个插件主要处理两部分逻辑

* 注入HMR runtime逻辑

* 找到修改的模块，生成一个补丁js文件和更新描述json文件

HMR runtime主要定义了jsonp 
callback方法，这个方法会触发模块更新，并且对模块新增一个module.hot相关API，这个API可以让开发者自定义页面更新逻辑。

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

### 服务器推送

文件更新后，首先需要打包好新的补丁文件，还需要告诉游览器文件修改了，可以拉代码了。

这一部分webpack自带了一个dev-server。当开启热加载的时候，webpack-dev-server会响应客户端发起的EventStream请求，然后保持请求不断开。这样服务器就可留意在有更新的时候直接把结果push到游览器。

服务器推送部分比较简单，构建一个node的Sever-Sent Events服务器只需要几行代码，这里有一个例子。

一次完成的构建流程大概是这样的

![png1]

上述步骤完成，热加载前两步就ok了。每次文件修改，游览器模块代码也更新了。但是就这样而言，模块更新还算不上完整的热加载，因为模块更新了，页面还没更新。前面提到构建过程中会在入口文件中加入一段HRM runtime代码，其中就有加上module.hot相关API。这个API就是提供给开发者自定义页面更新用的。

下面，我们进入热加载最后一步，页面局部更新。

runtime（运行时系统），是一套基于C语言API，包含在<objc/runtime.h>和<objc/message.h>中，运行时系统的功能是在运行期间（而不是编译期或其他时机）通过代码去动态的操作类（获取类的内部信息和动态操作类的成员），如创建一个新类、为某个类添加一个新的方法或者为某个类添加添加实例变量、属性，或者交换两个方法的实现、获取类的属性列表、方法列表等和Java中的发射技术类似。

### vue 热加载

据我了解vue热加载主要是用[vue-hot-reload-api](https://www.npmjs.com/package/vue-hot-reload-api)来实现。

众所周知，*.vue文件为广大开发者提供了良好的开发体验，vue-loader的原理不多赘述，在vue的脚手架中，webpack通过vue-loader来解析*.vue文件，把template、js和style文件分离并让相应的loader去处理。

在这个过程中，vue-loader还会做些其他事情，比如向client端注入hot-reload相应的代码，构建时编译等等。

webpack的hmr原理也不多说了，vue的热加载就是通过注入的代码来实现组件的热更新，下面来看下使用时的文档和源码。

先来看下官方文档。

> 你仅会在开发一个基于 Vue components 构建工具的时候用到这个。对于普通的应用，使用 vue-loader 或者 vueify 就可以了。
文档中明确说明了，一般使用不需要用到这个，只有在开发相应的构建工具时才会用到。

    // 定义一个组件作为选项对象
    // 在vue-loader中，这个对象是Component.options
    const myComponentOptions = {
      data () { ... },
      created () { ... },
      render () { ... }
    }
    
    // 检测 Webpack 的 HMR API
    // https://doc.webpack-china.org/guides/hot-module-replacement/
    if (module.hot) {
      const api = require('vue-hot-reload-api')
      const Vue = require('vue')
    
      // 将 API 安装到 Vue，并且检查版本的兼容性
      api.install(Vue)
    
      // 在安装之后使用 api.compatible 来检查兼容性
      if (!api.compatible) {
        throw new Error('vue-hot-reload-api与当前Vue的版本不兼容')
      }
    
      // 此模块接受热重载
      // 在这儿多说一句，webpack关于hmr的文档实在是太。。。
      // 各大框架的loader中关于hmr的实现都是基于自身模块接受更新来实现
      module.hot.accept()
    
      if (!module.hot.data) {
        // 为了将每一个组件中的选项变得可以热加载，
        // 你需要用一个不重复的id创建一次记录，
        // 只需要在启动的时候做一次。
        api.createRecord('very-unique-id', myComponentOptions)
      } else {
        // 如果一个组件只是修改了模板或是 render 函数，
        // 只要把所有相关的实例重新渲染一遍就可以了，而不需要销毁重建他们。
        // 这样就可以完整的保持应用的当前状态。
        api.rerender('very-unique-id', myComponentOptions)
    
        // --- 或者 ---
    
        // 如果一个组件更改了除 template 或 render 之外的选项，
        // 就需要整个重新加载。
        // 这将销毁并重建整个组件（包括子组件）。
        api.reload('very-unique-id', myComponentOptions)
      }
    }

通过使用说明可以看出，vue-hot-reload-api暴露的接口还是很清晰的，下面来看下具体源码实现。

    var Vue // late bind
    var version
    
    // 全局对象__VUE_HOT_MAP__来保存所有的构造器和实例
    var map = window.__VUE_HOT_MAP__ = Object.create(null)
    var installed = false
    
    // 这个参数来判断是vue-loader还是vueify在调用
    var isBrowserify = false
    
    // 2.0.0-alpha.7版本前的初始化钩子名是init，这个参数来作区分
    var initHookName = 'beforeCreate'
    
    exports.install = function (vue, browserify) {
      if (installed) return
      installed = true
    
    // 判断打包的是esodule还是普通的js函数
      Vue = vue.__esModule ? vue.default : vue
      version = Vue.version.split('.').map(Number)
      isBrowserify = browserify
    
      // compat with < 2.0.0-alpha.7
      if (Vue.config._lifecycleHooks.indexOf('init') > -1) {
        initHookName = 'init'
      }
    
      exports.compatible = version[0] >= 2
      // 兼容性，1.x和2.x的框架实现和loader实现都有很大差异
      if (!exports.compatible) {
        console.warn(
          '[HMR] You are using a version of vue-hot-reload-api that is ' +
          'only compatible with Vue.js core ^2.0.0.'
        )
        return
      }
    }
    
    /**
     * Create a record for a hot module, which keeps track of its constructor
     * and instances
     *
     * @param {String} id
     * @param {Object} options
     */
    
    exports.createRecord = function (id, options) {
      var Ctor = null
      // 判断传入的options是对象还是函数
      if (typeof options === 'function') {
        Ctor = options
        options = Ctor.options
      }
      // 燥起来，这个函数会在组件初始化和结束时的生命周期注入hook函数
      // 当实例化以后，hook函数调用会把实例记录到map中
      // destroy后会从map中删除实例自身
      makeOptionsHot(id, options)
      
      map[id] = {
        Ctor: Vue.extend(options),
        instances: []
      }
    }
    
    /**
     * Make a Component options object hot.
     *
     * @param {String} id
     * @param {Object} options
     */
    
    function makeOptionsHot (id, options) {
    // 注入hook函数，到达相应声明周期后执行
      injectHook(options, initHookName, function () {
        map[id].instances.push(this)
      })
      injectHook(options, 'beforeDestroy', function () {
        var instances = map[id].instances
        instances.splice(instances.indexOf(this), 1)
      })
    }
    
    /**
     * Inject a hook to a hot reloadable component so that
     * we can keep track of it.
     *
     * @param {Object} options
     * @param {String} name
     * @param {Function} hook
     */
    
    function injectHook (options, name, hook) {
    // 判断未注入时，生命周期init/beforeDestroy是否已经有了函数
    // 不存在的话，直接把生命周期函数置为[hook]
    // 存在的话，判断是否为Array，从而把已存在的函数和hook连接起来
      var existing = options[name]
      options[name] = existing
        ? Array.isArray(existing)
          ? existing.concat(hook)
          : [existing, hook]
        : [hook]
    }
    
    // 不得不说，这个一开始确实没搞懂是为啥要包一层
    // 自己实现的时候才知道，当有error弹出时
    // 如果不手动这样接住error，webpack会接到然后立即location.reload()
    // 根本来不及看reload之前给出的提示
    // 所以要手动处理下error
    function tryWrap (fn) {
      return function (id, arg) {
        try { fn(id, arg) } catch (e) {
          console.error(e)
          console.warn('Something went wrong during Vue component hot-reload. Full reload required.')
        }
      }
    }
    
    exports.rerender = tryWrap(function (id, options) {
      var record = map[id]
      // 边界处理
      // 如果没有传options或者已经为空
      // 会把这个构造函数生成的所有实例强制刷新并返回
      if (!options) {
        record.instances.slice().forEach(function (instance) {
          instance.$forceUpdate()
        })
        return
      }
      // 判断是否是构造函数还是proto
      if (typeof options === 'function') {
        options = options.options
      }
      
      // 修改map对象中的Ctor以便记录
      record.Ctor.options.render = options.render
      record.Ctor.options.staticRenderFns = options.staticRenderFns
      // .slice方法保证了instances的length是有效的
      record.instances.slice().forEach(function (instance) {
        // 把更新过的模块render函数和静态方法指到旧的实例上
        // reset static trees
        // 然后重刷新
        instance.$options.render = options.render
        instance.$options.staticRenderFns = options.staticRenderFns
        instance._staticTrees = [] // reset static trees
        instance.$forceUpdate()
      })
    })
    
    exports.reload = tryWrap(function (id, options) {
      var record = map[id]
      if (options) {
        if (typeof options === 'function') {
          options = options.options
        }
        makeOptionsHot(id, options)
        if (version[1] < 2) {
          // preserve pre 2.2 behavior for global mixin handling
          record.Ctor.extendOptions = options
        }
        
        // 其实最开始的commit中，并未继承Ctor的父类，是直接Vue.extend(options)
        // 对vue了解不深，不知道为啥改成这样
        // 有兴趣的同学可以思考下
        var newCtor = record.Ctor.super.extend(options)
        record.Ctor.options = newCtor.options
        record.Ctor.cid = newCtor.cid
        record.Ctor.prototype = newCtor.prototype
        // 2.0早期版本兼容
        if (newCtor.release) {
          // temporary global mixin strategy used in < 2.0.0-alpha.6
          newCtor.release()
        }
      }
      record.instances.slice().forEach(function (instance) {
      // 判断vNode和上下文是否存在
      // 不存在的需要手动刷新
        if (instance.$vnode && instance.$vnode.context) {
          instance.$vnode.context.$forceUpdate()
        } else {
          console.warn('Root or manually mounted instance modified. Full reload required.')
        }
      })
    })

短短的100多行代码，从这个库支持2.x的第一个commit读起，慢慢由简单实现到覆盖大部分边界及兼容性考虑，再到vue-loader的调用，webpack的hmr各种坑和debug，这个过程很受启发。

以为目前的水平，后半部分读起来有点困难，虽然有注释。。。之后再看

### css hot loader

js热加载基本上是通过自动更新组件，重新渲染页面两个步骤完成了。还有一个比较重要的是css热加载，webpack官方提供的方案是style-loader。

一般的对css处理都是通过extract-text-webpack-plugin插件把css抽离到单独css文件中。但extract-text-webpack-plugin是不支持热加载的，所以css热加载需要两个步骤：

 开发环境关闭extract-text-webpack-plugin
 开启sytle-loader插件

 style-loader实际上就是通过js创建一个style标签，然后注入内联的css。因为css是内联，并且通过js注入，那么页面刷新的时候一开始是没有任何css的，这种体验会非常差，闪一下然后页面重新渲染成功。

 webpack中扩展功能有两种方式

* loader
* plugin

 一个loader是对模块进行处理，比如css处理过程可以用这样来描述

    style-loader!css-loader!less

对每个css模块会依次执行less、css-loader、style-loader处理，每个loader处理后的结果作为下一个loader的输入字符串。

插件处理的是chunk，loader处理完成后，可以得到一个依赖树，每个模块都有一个处理结果的描述。在插件里面可以对整个entry输出的内容进行一些处理，比如热加载过程中增加HRM runtime脚本，对所有的css抽离到单独的静态资源中。

css-hot-loader所做的是在css模块中注入一段脚本，所以是一个loader，并且是第一个loader，这样可以保证代码不会被extract-text-webpack-plugin抽出来。

### 总结





参考http://shepherdwind.com/2017/02/07/webpack-hmr-principle/





[png1]: {{ site.baseurl }}/assets/images/20180211-1.svg